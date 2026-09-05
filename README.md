# Lab 08: Securing Your Software Supply Chain

## Aim
To secure the software supply chain by utilizing cryptographic signing to verify that a container image is exactly what was built, hasn't been tampered with, and is safe for production deployment.

---

## Architecture Diagram

```mermaid
graph TD
    subgraph Development Environment
        DEV[Developer Workstation]
        DOCKER[Docker Daemon]
        CODE[Dockerfile / App Code]
    end

    subgraph Cryptographic Signing
        COSIGN[Sigstore Cosign]
        KEY_PRIV[Private Key cosign.key]
        KEY_PUB[Public Key cosign.pub]
    end

    subgraph Production / CD Pipeline
        VERIFY[Verification Gate]
        PROD[Production Cluster]
    end

    DEV -->|1. Write Code| CODE
    CODE -->|2. Build Image| DOCKER
    
    DEV -->|3. Generate Keys| COSIGN
    COSIGN -.-> KEY_PRIV
    COSIGN -.-> KEY_PUB
    
    DOCKER -->|4. Unsigned Image| COSIGN
    KEY_PRIV -->|5. Sign Image| COSIGN
    
    COSIGN -->|6. Signed Image test-image:v1| VERIFY
    KEY_PUB -->|7. Verify Authenticity| VERIFY
    
    VERIFY -->|8. Validation Passed| PROD
    VERIFY -.->|Validation Failed| BLOCK[Block Deployment]
```

---

## Tools Required
* **Docker CLI:** Used to construct the container image from the source Dockerfile.
* **Sigstore Cosign:** An open-source tool to create digital signatures for container images and enforce supply-chain integrity.

---

## Execution Steps

### 1. Keypair Generation
1. Initialize the cryptographic keypair using Cosign:
   ```bash
   cosign generate-key-pair
   ```
2. Securely store the generated `cosign.key` (private) and `cosign.pub` (public) files.

### 2. Container Build and Signing
1. Build the local test container image using Docker:
   ```bash
   docker build -t test-image:v1 .
   ```
2. Sign the built image using the generated private key to establish a digital chain of custody:
   ```bash
   cosign sign --key cosign.key test-image:v1
   ```

### 3. Authenticity Verification
1. Validate the container image using the public key prior to simulated deployment:
   ```bash
   cosign verify --key cosign.pub test-image:v1
   ```
2. Confirm the CLI output states that the signatures were verified against the public key and that the cosign claims are valid.

---

## Screenshots

### 1. Keypair Generation & Image Signing
<img width="863" height="377" alt="image" src="https://github.com/user-attachments/assets/3a694f3a-46f2-458d-bab2-76ca2d8646fd" />

### 2. Image Verification Success
<img width="1866" height="857" alt="image" src="https://github.com/user-attachments/assets/c371f733-90fb-4303-b6c3-eddee899f67b" />



## Result
* Successfully generated cryptographic keypairs to act as a digital seal for software packages.
* Successfully built and signed a container image, proving its origin and integrity.
* Successfully verified the container image using its public key, demonstrating how to prevent tampered or unapproved images from reaching production environments.
