# 🔐 GPG Commit Signing Across Multiple Devices: A Complete Guide

This guide walks through using GPG to sign Git commits across multiple devices, integrating with GitLab, Ansible, and VS Code. It supports both **single-key** and **per-device key** approaches.

---

## 🧐 Key Management Strategy

### Option 1: Single GPG Key for All Devices

- ✅ Easier to manage and revoke
- ✅ GitLab only needs one key
- ❌ Must distribute private key to all machines

### Option 2: One GPG Key Per Device

- ✅ Device compromise does not affect others
- ✅ Easier to revoke access to one device
- ❌ More complex: GitLab needs multiple keys
- ❌ Commit signatures vary (but still show “Verified”)

**Recommendation:** Start with **single key** unless you need strict isolation.

---

## 🛠️ Setup Instructions

### Step 1: Generate a GPG Key

```bash
gpg --full-generate-key
```

- Key type: `RSA and RSA` or `ed25519` (via expert mode)
- Key size: 4096 for RSA, or use Curve 25519
- Set your Git name and email
- Set an expiration (recommended: 1–2 years)
- Use a strong passphrase

View the key:

```bash
gpg --list-secret-keys --keyid-format LONG
```

---

### Step 2: Export Your Key

#### Private Key (for distributing)

```bash
gpg --armor --export-secret-keys your@email.com > privatekey.asc
```

#### Public Key (for GitLab)

```bash
gpg --armor --export your@email.com > publickey.asc
```

---

### Step 3: Import Key on Other Devices (if using single-key setup)

Transfer `privatekey.asc` securely (USB, scp, 1Password attachment). Then:

```bash
gpg --import privatekey.asc
```

---

### Step 4: Add Public Key to GitLab

- Navigate to GitLab → **Preferences** → **GPG Keys**
- Paste in the contents of `publickey.asc`
- GitLab will show “Verified” on signed commits using that key

---

### Step 5: Configure Git

```bash
git config --global user.signingkey <YOUR_KEY_ID>
git config --global commit.gpgsign true
git config --global user.email "your@email.com"
git config --global user.name "Your Name"
```

---

### Step 6: Configure VS Code

`settings.json`:

```json
{
  "git.enableCommitSigning": true
}
```

VS Code uses system Git, so no extra extension is needed.

---

### Step 7: Use GPG Agent

Linux:

```bash
sudo apt install gnupg pinentry-gtk2
```

macOS:

- Install [GPG Suite](https://gpgtools.org)

Test it:

```bash
echo "hello" | gpg --clearsign
```

---

### Step 8: Test It

In a repo:

```bash
git commit -S -m "test: signed commit"
git log --show-signature
```

On GitLab, it should say **Verified**.

---

## 🛡️ Best Practices

- Always set a strong passphrase.
- Use `ansible-vault encrypt privatekey.asc` to secure it.
- Create a revocation certificate:
  ```bash
  gpg --output revoke.asc --gen-revoke <keyid>
  ```
- Backup your key and revocation cert offline (encrypted USB or password manager).

---

## 🧠 Pro Tips

- Use `git config --list` to confirm config.
- Each device can have its own key if needed—just import all public keys to GitLab.
- To sign one commit only:
  ```bash
  git commit -S -m "msg"
  ```
- Want hardware-backed keys? Try [Yubikey + GPG](https://github.com/drduh/YubiKey-Guide).

---

## 🛠️ Ansible Playbook: Distribute GPG Key to Multiple Devices

### Step 1: Export Your Private Key

```bash
gpg --armor --export-secret-keys your@email.com > privatekey.asc
```

### Step 2: Encrypt the Key File with Ansible Vault (Recommended)

```bash
ansible-vault encrypt privatekey.asc
```

---

### Step 3: Ansible Playbook: `distribute_gpg_key.yml`

```yaml
---
- name: Distribute and import GPG private key
  hosts: dev_machines
  become: false
  vars:
    gpg_key_file: privatekey.asc

  tasks:
    - name: Copy GPG private key to remote host
      ansible.builtin.copy:
        src: "{{ gpg_key_file }}"
        dest: "/tmp/privatekey.asc"
        owner: "{{ ansible_user }}"
        mode: '0600'
      no_log: true

    - name: Import GPG private key
      ansible.builtin.shell: |
        gpg --batch --yes --import /tmp/privatekey.asc
      args:
        executable: /bin/bash

    - name: Delete private key from remote host after import
      ansible.builtin.file:
        path: "/tmp/privatekey.asc"
        state: absent

    - name: List GPG secret keys (for confirmation)
      ansible.builtin.shell: |
        gpg --list-secret-keys --keyid-format LONG
      register: gpg_keys

    - name: Show imported key info
      ansible.builtin.debug:
        msg: "{{ gpg_keys.stdout_lines }}"
```

---

### Step 4: Run the Playbook

```bash
ansible-playbook -i hosts distribute_gpg_key.yml --ask-vault-pass
```

Ensure your inventory contains all dev machines under `[dev_machines]`.

---

### Optional: Public Key Only Playbook

```yaml
- name: Distribute public GPG key
  hosts: all
  tasks:
    - name: Copy public key
      copy:
        src: "publickey.asc"
        dest: "~/.gnupg/publickey.asc"
        mode: '0644'

    - name: Import public key
      shell: |
        gpg --import ~/.gnupg/publickey.asc
```

---

## ✅ You're Done!

You now have a secure and scalable setup to sign Git commits across your dev machines using GPG, with GitLab and VS Code integration, and automated deployment using Ansible.

Happy hacking!
