# CID_CRYPTOR
## Presentation
![image](./1.png)

**CID CRYPTOR** is a proprietary file encryption tool consisting of a Qt-based desktop application and a dedicated STM32H753 cryptographic module. The desktop application provides the user interface and communicates with the STM32H753 over USART, while all cryptographic operations are executed on the microcontroller. Consequently, cryptographic keys remain confined to the module and are never exposed to the host computer.

The cryptographic algorithms are implemented in software using ![Monocypher](https://monocypher.org/), a lightweight, open-source cryptographic library designed for portability, security, and ease of integration. The firmware performs authenticated encryption and decryption using the ChaCha20-Poly1305 AEAD (Authenticated Encryption with Associated Data) algorithm.

By isolating cryptographic processing within the STM32H753, CID_CRYPTOR provides secure key handling while protecting sensitive files. The use of ChaCha20-Poly1305 ensures confidentiality, integrity, and authenticity, allowing any unauthorized modification of an encrypted file to be detected during decryption.

## Standard user features
### Connection 
Before using **CID CRYPTOR**, the user must connect the desktop application to the cryptographic module. Once connected, the connection can be verified by sending a **Ping** command to ensure that the device is responding correctly. The application also allows the user to retrieve information about the connected module.
![Device connection and information](./2.png)

## Random Number Generation

The cryptographic module uses the **True Random Number Generator (TRNG)** integrated into the STM32H753 microcontroller to generate cryptographically secure random data. These random bytes can be used for security-critical operations such as generating **nonces**, **salts**, or seeding a **Pseudo-Random Number Generator (PRNG)**.

![Random number generation](./3.png)

The module can generate up to 1MiB of random data.

![Random number generation](./3_1.png)

## Key recovery

![key recovery](./4.png)

## Key Management

The system uses two types of cryptographic keys:

- **Session keys**, which are used to perform encryption and decryption operations.
- A **master key**, which is used exclusively to encrypt and decrypt the session keys.

The module provides **eight session key slots**, allowing different keys to be assigned to different applications or security domains. Session keys are **never stored in plaintext**; instead, they are encrypted using the master key before being written to non-volatile memory.

Unlike a conventional stored key, the **master key is never permanently stored**. Instead, it is **reconstructed on demand** from three independent components:

1. The microcontroller's unique hardware identifier (**UUID**), stored in the STM32H753's registers.
2. A device secret stored in a protected region of the internal Flash memory.
3. A user-provided password entered during the key recovery process.

Only when all three components are available can the master key be reconstructed to decrypt a session key. Once recovered, the session key remains in RAM only for a configurable period defined by the **session timeout** parameter. After the timeout expires, the plaintext session key is securely erased from memory and must be recovered again before it can be used.

This architecture significantly strengthens the security of the system. An attacker attempting to recover a session key would need to obtain all three components required to reconstruct the master key. In practice, this means the attacker must have **physical access to the cryptographic module**, extract the protected device secret, and know the **user's password**. Without all three components, the encrypted session keys remain unusable.



