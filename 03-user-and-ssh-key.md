# 03 — Your own user and an SSH key

The door is SSH, and right after the install `root` opens it with just a password. A password can be guessed or copied; an SSH key can't be guessed.

## 1. Create your user (on the server, as root)

From the web UI **Shell** (`pve` → **Shell**) or your current SSH session:

```bash
adduser itslucho               # create the user; use a long password (you'll only need it for sudo)
apt install sudo               # Proxmox doesn't ship with sudo: it's meant to be managed as root
usermod -aG sudo itslucho      # add the user to the group that can use sudo
```

Replace `itslucho` with your own user name.

## 2. Copy your public key to the server (from your computer)

### My case: YubiKey with OpenPGP (via `gpg-agent`)

My SSH key is the **authentication subkey** of the OpenPGP key stored on my YubiKey 5C NFC. The private key never leaves the YubiKey; `gpg-agent` acts as the SSH agent.

```bash
ssh-add -L                                # show the public key the agent has (the YubiKey one)
ssh-copy-id itslucho@pve.itslucho.com     # copy it to ~/.ssh/authorized_keys of your user on the server
ssh itslucho@pve.itslucho.com             # test: touch the YubiKey when it blinks
```

`ssh-copy-id` without `-i` copies the keys the agent has (`ssh-add -L`). It asks for your user's password one last time.

Setting up a YubiKey with OpenPGP for SSH: [Yubico — Using Your YubiKey with OpenPGP](https://support.yubico.com/hc/en-us/articles/360013790259-Using-Your-YubiKey-with-OpenPGP).

### Alternative: FIDO2 key (`ed25519-sk`)

Works with any FIDO2 security key. **On macOS**, the built-in OpenSSH doesn't support FIDO2 keys: install OpenSSH with Homebrew first (`brew install openssh`).

```bash
ssh-keygen -t ed25519-sk -C "my-security-key"
ssh-copy-id -i ~/.ssh/id_ed25519_sk.pub itslucho@pve.itslucho.com
```

### Alternative: a normal key file (no hardware)

```bash
ssh-keygen -t ed25519 -C "my-laptop"
ssh-copy-id -i ~/.ssh/id_ed25519.pub itslucho@pve.itslucho.com
```

## 3. Have a backup

If the only authorized key lives on one YubiKey and you lose it, you lose SSH access (the web UI keeps working). I have **the same key on a second YubiKey**, stored away. Other options: register two keys in `authorized_keys`, or keep a normal key file stored offline.

**Don't turn off password logins until you have logged in with the key.** That's the next step: [04-sshd-config.md](04-sshd-config.md).
