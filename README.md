# StegoCrypt-Secure-Data-Hiding-Using-Cryptography-and-Steganography
# Procedure :
Step 1 — Create Project Directory
```
mkdir stegocrypt
cd stegocrypt
```
Step 2 — Create Python File
```
nano stegocrypt.py
```

```
from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import padding
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes
import os
from PIL import Image
import base64

# =========================
# KEY GENERATION
# =========================
def generate_key(password, salt):
    kdf = PBKDF2HMAC(
        algorithm=hashes.SHA256(),
        length=32,
        salt=salt,
        iterations=100000,
        backend=default_backend()
    )
    return kdf.derive(password.encode())

# =========================
# ENCRYPTION
# =========================
def encrypt_message(key, plaintext):
    iv = os.urandom(16)

    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
    encryptor = cipher.encryptor()

    padder = padding.PKCS7(128).padder()
    padded = padder.update(plaintext.encode()) + padder.finalize()

    ciphertext = encryptor.update(padded) + encryptor.finalize()
    return iv + ciphertext

# =========================
# DECRYPTION
# =========================
def decrypt_message(key, data):
    iv = data[:16]
    ciphertext = data[16:]

    cipher = Cipher(algorithms.AES(key), modes.CBC(iv), backend=default_backend())
    decryptor = cipher.decryptor()

    padded = decryptor.update(ciphertext) + decryptor.finalize()

    unpadder = padding.PKCS7(128).unpadder()
    plaintext = unpadder.update(padded) + unpadder.finalize()

    return plaintext.decode()

# =========================
# IMAGE FORMAT HANDLER
# =========================
def prepare_image(image_path):
    ext = image_path.lower().split('.')[-1]

    if ext in ['jpg', 'jpeg']:
        print("[!] Converting JPG → PNG...")
        img = Image.open(image_path)
        new_path = "converted_input.png"
        img.save(new_path, "PNG")
        return new_path
    elif ext == 'png':
        return image_path
    else:
        raise Exception("Use JPG, JPEG or PNG")

# =========================
# EMBED MESSAGE
# =========================
def embed_message(image_path, data, output_path):
    image_path = prepare_image(image_path)

    img = Image.open(image_path)

    if img.mode != 'RGB':
        img = img.convert('RGB')

    encoded = img.copy()
    width, height = img.size

    b64_data = base64.b64encode(data)
    data_len = len(b64_data)

    full_data = format(data_len, '032b') + ''.join(format(b, '08b') for b in b64_data)

    if len(full_data) > width * height:
        raise Exception("Image too small!")

    index = 0

    for row in range(height):
        for col in range(width):
            if index < len(full_data):
                pixel = list(encoded.getpixel((col, row)))
                pixel[0] = pixel[0] & ~1 | int(full_data[index])
                encoded.putpixel((col, row), tuple(pixel))
                index += 1
            else:
                encoded.save(output_path)
                print(f"[+] Saved as {output_path}")
                return

# =========================
# EXTRACT MESSAGE
# =========================
def extract_message(image_path):
    img = Image.open(image_path)

    if img.mode != 'RGB':
        img = img.convert('RGB')

    width, height = img.size
    bits = ""

    for row in range(height):
        for col in range(width):
            pixel = list(img.getpixel((col, row)))
            bits += str(pixel[0] & 1)

    data_len = int(bits[:32], 2)
    data_bits = bits[32:32 + data_len * 8]

    b64_bytes = bytearray()

    for i in range(0, len(data_bits), 8):
        b64_bytes.append(int(data_bits[i:i+8], 2))

    return base64.b64decode(b64_bytes)

# =========================
# OPTION 1: ENCRYPT + EMBED
# =========================
def hide_message():
    image_input = input("Enter image path: ")
    password = input("Enter password: ")
    message = input("Enter secret message: ")

    salt = os.urandom(16)
    key = generate_key(password, salt)

    print("\n[+] Encrypting...")
    encrypted = encrypt_message(key, message)

    final_data = salt + encrypted

    print("[+] Embedding into image...")
    embed_message(image_input, final_data, "output_image.png")

    print("✅ Message hidden successfully!\n")

# =========================
# OPTION 2: EXTRACT + DECRYPT
# =========================
def extract_and_decrypt():
    image_input = input("Enter stego image path: ")
    password = input("Enter password: ")

    print("\n[+] Extracting...")
    extracted = extract_message(image_input)

    salt = extracted[:16]
    encrypted = extracted[16:]

    key = generate_key(password, salt)

    print("[+] Decrypting...")
    try:
        message = decrypt_message(key, encrypted)
        print("\n✅ Secret Message:", message)
    except:
        print("\n❌ Wrong password or corrupted image")

# =========================
# MAIN MENU
# =========================
def main():
    while True:
        print("\n====== STEGOCRYPT TOOL ======")
        print("1. Hide Message")
        print("2. Extract Message")
        print("3. Exit")

        choice = input("Enter choice: ")

        if choice == '1':
            hide_message()
        elif choice == '2':
            extract_and_decrypt()
        elif choice == '3':
            print("Exiting...")
            break
        else:
            print("Invalid choice!")

if __name__ == "__main__":
    main()
```

Step 3 — Add Input Image

Place an image inside the folder:

input_image.png

Supported formats:

PNG ✅ (recommended)
JPG / JPEG ✅ (auto converted)

Step 4 — Run the Program

python3 stegocrypt.py

Step 5 — Choose Operation

====== STEGOCRYPT TOOL ======
1. Hide Message
2. Extract Message
3. Exit
🔐 Hiding Message (Encryption + Embedding)

Step 1

Select:

1

Step 2

Enter details:

Enter image path: input.jpg
Enter password: mypass123
Enter secret message: Hello World
Step 3 — Processing
[!] Converting JPG → PNG...
[+] Encrypting...
[+] Embedding into image...
[+] Saved as output_image.png

✅ Message hidden successfully!
Output File
output_image.png

👉 This image contains:

Encrypted message
Salt
Hidden data in pixels
🔓 Extracting Message (Decryption)
Step 1

Select:

2
Step 2

Enter details:

Enter stego image path: output_image.png
Enter password: mypass123
Step 3 — Output
[+] Extracting...
[+] Decrypting...

✅ Secret Message: Hello World

# Output:

<img width="1134" height="444" alt="image" src="https://github.com/user-attachments/assets/ce3d215b-a38e-4179-9108-9451616ceed0" />

<img width="556" height="348" alt="image" src="https://github.com/user-attachments/assets/5e870143-bbf1-4a0d-bd52-61812b0c6460" />

<img width="1053" height="385" alt="image" src="https://github.com/user-attachments/assets/93a7783e-e672-4353-bb37-b9b2fae141f1" />

<img width="561" height="573" alt="image" src="https://github.com/user-attachments/assets/65090232-f9d7-4af2-a512-2fc6eeda4ee4" />



