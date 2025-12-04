# Experimental Code: TLS 1.3 Handshake with Hybrid Key Exchange from CPA-Secure KEMs

This repository contains the implementation and testing code for our paper **"On the Security and Efficiency of TLS 1.3 Handshake with Hybrid Key Exchange from CPA-Secure KEMs"**.

The project enables reproduction of the performance evaluation experiments comparing CCA-secure and CPA-secure ML-KEM implementations within the TLS 1.3 handshake protocol.

## 🛠️ System Requirements

- **Operating System**: Ubuntu 24.04 LTS
- **Processor**: 64-bit CPU (x86_64 or ARM architecture)
- **Python**: Version 3.12.3

## 📥 OpenSSL Installation & Configuration
The experiments were conducted using OpenSSL 3.6.0 (released October 1, 2025). Follow these steps to compile and install it from source (installation guide reference: https://www.yisu.com/ask/98212499.html. Credit and thanks to the original author):

### Prerequisites
```bash
sudo apt update
sudo apt install build-essential checkinstall zlib1g-dev -y
```

### Build & Install OpenSSL 3.6.0

#### Download and extract
```bash
wget https://github.com/openssl/openssl/releases/download/openssl-3.6.0/openssl-3.6.0.tar.gz
tar -xzvf openssl-3.6.0.tar.gz
cd openssl-3.6.0
```

#### Configure and compile
```bash
./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl shared zlib
make
sudo make install
```

#### Update library paths
```bash
echo "/usr/local/openssl/lib64" | sudo tee -a /etc/ld.so.conf.d/openssl.conf
sudo ldconfig
```

#### Add to PATH
```bash
export PATH=/usr/local/openssl/bin:$PATH
```
You can add this line to your `~/.bashrc` or `~/.profile` file to have it configured automatically each time you open a new terminal.

### Verification
```bash
openssl version
# Expected output: OpenSSL 3.6.0 1 Oct 2025 (Library: OpenSSL 3.6.0 1 Oct 2025)
```

## 🔄 CPA-Secure ML-KEM Implementation

OpenSSL uses a CCA-secure ML-KEM implementation based on the FIPS 203 standard. To test with the CPA-secure version:

1. **Backup** the original `ml_kem.c` in `/openssl-3.6.0/crypto/ml_kem/`.

2. **Replace** the original `ml_kem.c` in `/openssl-3.6.0/crypto/ml_kem/` with the CPA-secure version `ml_kem.c` provided in this repository.

3. **Recompile** OpenSSL by repeating Step **Configure and compile** from the installation instructions.

> **Note**: To revert to the CCA-secure version, simply restore the backup file and recompile.

## 📊 Performance Evaluation

### Algorithm-Level Benchmarking

Test raw cryptographic operations for each hybrid scheme:
```bash
openssl speed -seconds 3 <SCHEME>
```
Replace `<SCHEME>` with: `X25519MLKEM768`, `SecP256r1MLKEM768`, or `SecP384r1MLKEM1024`.
This command returns test results for key generation, encapsulation, and decapsulation operations over a 3-second period.

### Protocol-Level Benchmarking

#### 1. Generate Test Certificate
```bash
openssl ecparam -name prime256v1 -genkey -noout -out server.key
openssl req -new -x509 -sha256 -key server.key -out server.crt -days 365 -subj "/CN=localhost"
```

#### 2. Start TLS Server
In one terminal window:
```bash
openssl s_server -cert server.crt -key server.key -accept 4443 -www -groups <SELECTED_GROUPS>
```
Replace `<SELECTED_GROUPS>` with: `X25519MLKEM768`, `SecP256r1MLKEM768`, or `SecP384r1MLKEM1024`.

#### 3. Measure Handshake Performance
In another terminal window:
```bash
openssl s_time -connect localhost:4443 -new -time 3
```
This returns test results based on a 3-second measurement period.

## ⚙️ TLS Groups Configuration

By default, `s_time` may not include `SecP256r1MLKEM768` and `SecP384r1MLKEM1024` in its supported TLS groups. You can either:

1. **Replace** the existing `openssl.cnf` file at `/usr/local/openssl/` with the one provided in this repository, OR

2. **Manually configure** supported TLS groups as follows (reference: [https://openssl-library.org/post/2022-10-21-tls-groups-configuration/](https://openssl-library.org/post/2022-10-21-tls-groups-configuration/)):

#### Locate Configuration File
```bash
openssl version -d  # Output: OPENSSLDIR: "/usr/local/openssl"
```

#### Update `/usr/local/openssl/openssl.cnf`:
```ini
# Add if not present
openssl_conf = openssl_init

[openssl_init]
ssl_conf = ssl_module

[ssl_module]
system_default = tls_system_default

[tls_system_default]
Groups = X25519MLKEM768:SecP256r1MLKEM768:SecP384r1MLKEM1024
```

## 🤖 Automated Testing

To ensure statistical significance, we conducted 50 test iterations for each scheme under both CCA-secure and CPA-secure ML-KEM versions. Two Python scripts are provided for automated testing:

### Algorithm-Level Tests
```bash
python3 runtime_KE.py <SCHEME>
```
Replace `<SCHEME>` with: `X25519MLKEM768`, `SecP256r1MLKEM768`, or `SecP384r1MLKEM1024`.

### Protocol-Level Tests
```bash
python3 runtime_HS.py <SELECTED_GROUPS>
```
Replace `<SELECTED_GROUPS>` with: `X25519MLKEM768`, `SecP256r1MLKEM768`, or `SecP384r1MLKEM1024`.


## 📈 Results & Data

The scripts generate Excel files (as shown in the `data/x86_64` and `data/ARM` folders, containing raw experimental data used in this work).

> **Important**: The scripts cannot automatically detect the ML-KEM security version. The filename prefixes `CCA_` and `CPA_` were manually added after data generation to distinguish between CCA-secure and CPA-secure implementations.

All data files in the `data` folder represent the original measurements used in our paper analysis.

## 📚 Reference

For detailed experimental methodology and results analysis, please refer to the accompanying paper.
