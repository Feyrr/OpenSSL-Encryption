# OpenSSL Encryption Guide

This guide explains how to encrypt and decrypt files using OpenSSL.  
Two main methods are included:

1. **Password-based encryption** (PBKDF2 + AES-256-CBC)  
2. **Key-file-based encryption** (AES-256-CBC with raw key)

## 1️⃣ Password-Based Encryption

Password-based encryption uses a passphrase to derive the encryption key using PBKDF2.

### Step 1: Encrypt a file

```bash
# Encrypt using a password
openssl enc -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt -in plaintext.txt -out secret.enc
```

**Options explained:**

- `-aes-256-cbc` → Use AES cipher with 256-bit key in CBC mode  
- `-md sha512` → Use SHA-512 to derive the key from the password  
- `-pbkdf2` → Use the modern PBKDF2 key derivation function  
- `-iter 100000` → Number of PBKDF2 iterations (slows brute-force attacks)  
- `-salt` → Adds a random salt to strengthen the key derivation  
- `-in plaintext.txt` → Input plaintext file to encrypt  
- `-out secret.enc` → Output encrypted file  

### Step 2: Decrypt a file

```bash
openssl enc -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt -d -in secret.enc -out decrypted.txt
```

- Enter the **same password** used for encryption.

### Optional: Shell helper functions

```bash
function openssl-encrypt() {
    openssl enc -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt -in "$1" -out "$2"
}

function openssl-decrypt() {
    openssl enc -aes-256-cbc -md sha512 -pbkdf2 -iter 100000 -salt -d -in "$1" -out "$2"
}
```

Usage:

```bash
openssl-encrypt myfile.txt myfile.enc
openssl-decrypt myfile.enc myfile.txt
```

## 2️⃣ Key-File-Based Encryption

Key-file encryption uses a raw key and IV instead of a password.  
This is useful for automation and scripting.

### Step 1: Generate a random key and IV

```bash
# Generate 32-byte key for AES-256
openssl rand -out keyfile.bin 32

# Generate 16-byte IV for AES-CBC
openssl rand -out iv.bin 16
```

**Options explained:**

- `openssl rand -out keyfile.bin 32` → Generates 32 random bytes for the AES-256 key  
- `openssl rand -out iv.bin 16` → Generates 16 random bytes for the AES CBC IV  

### Step 2: Encrypt a file using the key file

```bash
openssl enc -aes-256-cbc -in plaintext.txt -out secret.enc \
    -K $(xxd -p -c 256 keyfile.bin) \
    -iv $(xxd -p -c 256 iv.bin)
```

**Options explained:**

- `-aes-256-cbc` → Use AES-256-CBC cipher  
- `-in plaintext.txt` → Input file to encrypt  
- `-out secret.enc` → Output encrypted file  
- `-K $(xxd -p -c 256 keyfile.bin)` → Use the raw key from keyfile (converted to hex)  
- `-iv $(xxd -p -c 256 iv.bin)` → Use the initialization vector from iv.bin (converted to hex)  

### Step 3: Decrypt a file using the key file

```bash
openssl enc -aes-256-cbc -d -in secret.enc -out decrypted.txt \
    -K $(xxd -p -c 256 keyfile.bin) \
    -iv $(xxd -p -c 256 iv.bin)
```

- Make sure to use the **same key** and **same IV** used for encryption.

### Optional: Shell helper functions for key-file encryption

```bash
function openssl-encrypt-key() {
    openssl enc -aes-256-cbc -in "$1" -out "$2" \
        -K $(xxd -p -c 256 keyfile.bin) \
        -iv $(xxd -p -c 256 iv.bin)
}

function openssl-decrypt-key() {
    openssl enc -aes-256-cbc -d -in "$1" -out "$2" \
        -K $(xxd -p -c 256 keyfile.bin) \
        -iv $(xxd -p -c 256 iv.bin)
}
```

Usage:

```bash
openssl-encrypt-key myfile.txt myfile.enc
openssl-decrypt-key myfile.enc myfile.txt
```

## ⚠️ Security Notes

1. **Protect your keyfile and IV.** Anyone with them can decrypt your files.  
2. **Use a random IV for each encryption** if encrypting multiple files. You can prepend the IV to the ciphertext and extract it during decryption.  
3. **Password-based encryption** with PBKDF2 is usually sufficient for most use cases. Key-file encryption is better for automation or scripts.  
4. Avoid supplying passwords directly in shell commands (`-pass pass:…`) to prevent exposure in shell history.
