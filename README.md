# Secure Password Handling in Bash Scripts (Using OpenSSL)

## 📌 Purpose

This document explains how to securely store and use passwords in Bash scripts without hardcoding them in plain text.

This method works for:

* Database passwords
* API keys
* SSH passphrases
* Encryption passwords
* Any sensitive credential

It is **not database-specific** and can be reused in any Bash automation.

---

# 🔐 Overview

We use:

* `openssl` for encryption
* A master key file
* Encrypted password files
* Runtime decryption inside scripts

The password is never stored in plain text inside the script.

---

# 📁 File Structure

Example:

```
/root/secure/
├── master.key
├── db_pass.enc
├── api_key.enc
├── backup_pass.enc
```

---

# 1️⃣ Create Master Key (One Time Only)

Generate a strong random key:

```bash
openssl rand -base64 64 > /root/secure/master.key
chmod 600 /root/secure/master.key
```

⚠️ Important:

* Do NOT commit this file to GitHub
* Backup this file securely
* If lost, encrypted files cannot be decrypted

---

# 2️⃣ Encrypt Any Password

Replace `YourSecretPassword` with your real password:

```bash
echo -n 'YourSecretPassword' | openssl enc -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt \
  -pass file:/root/secure/master.key \
  -out /root/secure/example_pass.enc
```

Secure it:

```bash
chmod 600 /root/secure/example_pass.enc
```

You can repeat this process for any number of passwords.

---

# 3️⃣ Decrypt Inside a Bash Script

Create a reusable decrypt function:

```bash
decrypt() {
  openssl enc -d -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt \
    -pass file:/root/secure/master.key \
    -in "$1"
}
```

Use it like this:

```bash
MY_PASSWORD=$(decrypt /root/secure/example_pass.enc)
```

Now you can safely use:

```bash
some_command --password "$MY_PASSWORD"
```

---

# 4️⃣ Example Full Script Usage

```bash
#!/bin/bash

KEY_FILE="/root/secure/master.key"
PASS_FILE="/root/secure/example_pass.enc"

decrypt() {
  openssl enc -d -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt \
    -pass file:"$KEY_FILE" \
    -in "$1" 2>/dev/null
}

PASSWORD=$(decrypt "$PASS_FILE")

if [ -z "$PASSWORD" ]; then
  echo "Decryption failed"
  exit 1
fi

echo "Password loaded securely"
```

---

# 🔒 Security Best Practices

### ✅ Always:

* Set permission to `600`
* Store key outside user home if possible
* Backup master key securely
* Restrict root access

### ❌ Never:

* Store master.key in Git
* Echo decrypted password in logs
* Send encrypted files without the key
* Hardcode passwords in scripts

---

# 🛡 Recommended Enhancements

For higher security environments:

* Store key on encrypted disk
* Use separate key per environment (dev/stage/prod)
* Rotate encrypted passwords periodically
* Use OS-level secret managers if available

---

# 🚨 Important Warning

If `master.key` is deleted or corrupted:

👉 All encrypted password files become useless permanently.

There is no recovery.

---

# 📌 Why This Method?

✔ Keeps passwords out of scripts
✔ Works in cron jobs
✔ Works in automation
✔ Simple and portable
✔ No external secret manager required

