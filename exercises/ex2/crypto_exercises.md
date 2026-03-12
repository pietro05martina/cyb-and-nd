# Cybersecurity and National Defence Ex. 2 
## A.Y. 2025/2026

### Contacts
* **Flavio Ciravegna:** [*flavio.ciravegna@polito.it*]
* **Silvia Sisinni:** [*silvia.sisinni@polito.it*]
* **Enrico Bravi:** [*enrico.bravi@polito.it*]
* **Lorenzo Ferro:** [*lorenzo.ferro@polito.it*]

---

## Agenda 

1. Virtual Environments
2. Bash Command Explanations
3. Operations with OpenSSL

---

## 1. Virtual Environments and Setup
Before installing external libraries (`pip install cryptography`), it is best practice to create a virtual environment.

### Windows
Run in Command Prompt or PowerShell:
```cmd
python -m venv myenv
myenv\Scripts\activate
```

### Mac/Linux
Run in your terminal:
```bash
python3 -m venv myenv
source myenv/bin/activate
```

### Config VSCode & Jupyter Notebook
If you are using Visual Studio Code to complete the Jupyter Notebook exercise, you must instruct VSCode to use your newly created virtual environment as the notebook kernel. 

1. Install the official **Jupyter** and **Python** extensions in VSCode.
2. Open the `crypto_tutorial.ipynb` notebook.
3. In the top right corner of the notebook interface, click on the **Select Kernel** button (it might say "Python 3" or similar).
4. Click **Python Environments** -> **Select another...** -> **Python Environments**.
5. Choose the interpreter located in your `./myenv/bin/python` (or `\myenv\Scripts\python.exe` on Windows).

---

### Bash Command Explanations

The command-line examples above make use of several fundamental Unix/Linux bash utilities to manage data inputs and outputs:

*   **`echo -n`**: The `echo` command prints text to the terminal. The `-n` flag tells it *not* to add a trailing newline character at the end of the string, which is critical for cryptography so we do not accidentally encrypt/hash a newline character.
*   **`|` (Pipe)**: The pipe operator takes the standard output (`stdout`) of the command on the left and literally "pipes" it as the standard input (`stdin`) into the command on the right. For example, `echo -n "..." | openssl base64` passes the raw text directly into the openssl engine without needing a temporary file.
*   **`>` (Redirection)**: The right-angle bracket redirects standard output into a file, creating the file if it doesn't exist or overwriting it if it does. For example, `echo -n "..." > plaintext.txt` saves the string to that text file. `-e` gives information on how to interpret some special characters
*   **`xxd -p`**: `xxd` is a command used to create a hex dump of a given file or standard input. The `-p` flag outputs this in a continuous "plain" hexdump formatting, making it easy to read encrypted binary streams as text characters.
*   **`cat`**: This command reads files sequentially and prints their contents directly to the standard output screen.

---

The exercises in the Jupyter Notebook can also be performed via the terminal using `openssl`. Here is a brief explanation and the equivalent command-line examples.

## 2. Base64 Encoding and Decoding
**What it is:** Base64 is an encoding scheme that translates raw binary data into a string of 64 specific ASCII characters. 
**Purpose:** It is not encryption, but a format change. It ensures that binary data (like images or encrypted files) can be safely transmitted over protocols that were designed only for text (like HTTP or SMTP).

**Encoding:**
```bash
echo -n "Secret message that needs to be encoded" | openssl base64
```

**Decoding:**
```bash
echo -e "U2VjcmV0IG1lc3NhZ2UgdGhhdCBuZWVkcyB0byBiZSBlbmNvZGVk\n" | openssl base64 -d
```

## 3. Hashing
**What it is:** A cryptographic hash function takes data of any size and produces a fixed-size, unique "fingerprint" (the hash) of that data. It is a one-way mathematical function meaning you cannot derive the original data from the hash.
**Purpose:** Hashing is used to verify data integrity. If even a single byte of a file changes, its resulting hash will change completely, letting you know the file has been tampered with. It is also used to store passwords securely.

**Hashing with SHA-256:**
```bash
echo -n "This is some data to hash" | openssl dgst -sha256
```

## 4. Symmetric Encryption
**What it is:** Symmetric encryption is a cipher technique that uses the *exact same* secret key to mathematically both encrypt and decrypt the data. AES (Advanced Encryption Standard) is the global standard for symmetric encryption.
**Purpose:** Symmetric encryption is extremely fast, making it ideal for encrypting large amounts of data (like files, databases, or entire hard drives). The main challenge is securely sharing the single secret key with the recipient. 

**Encrypting:**
```bash
# You will be prompted to enter a password to derive the encryption key
echo -n "This is highly classified symmetric data that we need to protect." > plaintext.txt
openssl enc -aes-256-cbc -pbkdf2 -iter 100000 -salt -in plaintext.txt -out encrypted.bin

# To view it as hex:
xxd -p encrypted.bin > symmetric_encrypted.txt
```

**Decrypting:**
```bash
openssl enc -d -aes-256-cbc -pbkdf2 -iter 100000 -in encrypted.bin -out decrypted.txt
cat decrypted.txt
```

## 5. Asymmetric Operations
**What it is:** Asymmetric encryption uses a pair of mathematically linked keys: a **public key** (which you can share with the world) and a **private key** (which you must carefully protect). RSA is the most famous asymmetric algorithm.
**Purpose:** It solves the key distribution problem of symmetric encryption. Anyone can use your public key to encrypt a message for you, but only you can decrypt it with your private key. It is the foundation of secure web browsing (TLS/SSL). 

### Key Pair Generation
Generate a 2048-bit RSA private key and extract its public key:
```bash
openssl genrsa -out private_key.pem 2048
openssl rsa -in private_key.pem -pubout -out public_key.pem
```

### Asymmetric Encryption
Encrypt data using the public key (only the private key holder can decrypt it):
```bash
echo -n "This secret is protected via Public Key infrastructure." > asym_message.txt
openssl pkeyutl -encrypt -in asym_message.txt -pubin -inkey public_key.pem -out asym_encrypted.bin
```

### Asymmetric Decryption
Decrypt data using the private key:
```bash
openssl pkeyutl -decrypt -in asym_encrypted.bin -inkey private_key.pem -out asym_decrypted.txt
cat asym_decrypted.txt
```

### Cryptographic Signing
**What it is:** Digital signatures are created by doing asymmetric operations in reverse: you "encrypt" a hash of a document using your **private key**. 
**Purpose:** Signing proves authenticity (that you are the true author of the file) and non-repudiation (you cannot later deny signing it) because only your specific private key could have produced that signature.
```bash
echo -n "This is a legally binding contract." > document_to_sign.txt
openssl dgst -sha256 -sign private_key.pem -out signature.bin document_to_sign.txt
```

### Signature Verification
Verify the document's signature using the public key:
```bash
openssl dgst -sha256 -verify public_key.pem -signature signature.bin document_to_sign.txt
```
