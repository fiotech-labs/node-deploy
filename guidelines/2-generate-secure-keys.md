# 2. Hướng Dẫn Tạo và Quản Lý Private Keys Bảo Mật

## Tổng Quan

Hướng dẫn này sẽ giúp bạn:

- Hiểu vấn đề bảo mật của keys hiện tại
- Tạo keys mới bảo mật hơn
- Quản lý keys một cách an toàn
- Implement security best practices

## ⚠️ Vấn Đề Bảo Mật Hiện Tại

### Keys Mặc Định Không An Toàn

Repository hiện tại có các vấn đề bảo mật nghiêm trọng:

```
❌ Password yếu: "0123456789"
❌ Keys được commit công khai trong git
❌ Node keys không mã hóa (plaintext hex)
❌ Có thể dự đoán được (ai clone repo cũng có keys)
❌ Không có key rotation
❌ Không có hardware security
```

### Tác Động

- **Development**: OK - chỉ dùng để test
- **Testnet**: NGUY HIỂM - có thể bị hack
- **Mainnet**: THỦ HỌNG SINH MẠNG - mất hết tài sản

## Giải Pháp Bảo Mật

### Cấp Độ 1: Development (Cơ bản)

- Tạo keys mới với password mạnh
- Không commit keys vào git
- Backup cơ bản

### Cấp Độ 2: Testnet (Trung bình)

- Tất cả yêu cầu của Development
- Multi-signature wallets
- Monitoring và alerting
- Regular key rotation

### Cấp Độ 3: Mainnet (Cao nhất)

- Tất cả yêu cầu của Testnet
- Hardware Security Modules (HSM)
- Air-gapped key generation
- Multi-location backup
- Professional security audit

## Cách Tạo Keys Bảo Mật (Thủ Công)

### Bước 1: Backup Keys Hiện Tại

```bash
# Backup keys cũ trước khi thay thế
BACKUP_DIR="keys-backup-$(date +%Y%m%d-%H%M%S)"
echo "Backing up existing keys to: $BACKUP_DIR"

if [ -d "keys/" ]; then
    cp -r keys/ "$BACKUP_DIR/"
    echo "✅ Backup completed: $BACKUP_DIR"
else
    echo "ℹ️  No existing keys directory found"
fi
```

### Bước 2: Tạo Password Mạnh

```bash
# Phương pháp 1: Tạo password random 25 ký tự
PASSWORD=$(openssl rand -base64 32 | tr -d "=+/" | cut -c1-25)
echo "Generated password: $PASSWORD"

# Phương pháp 2: Tạo passphrase dài (khuyến nghị cho production)
PASSWORD="correct-horse-battery-staple-mountain-ocean-forest-$(date +%s)"
echo "Generated passphrase: $PASSWORD"

# Tạo thư mục keys và lưu password
mkdir -p keys
echo "$PASSWORD" > keys/password.txt
echo "" >> keys/password.txt  # Add empty line like original format

echo "✅ Password saved to keys/password.txt"
echo "⚠️  QUAN TRỌNG: Lưu password này ở nơi an toàn!"
```

### Bước 3: Tạo Validator Accounts

```bash
# Kiểm tra số lượng validators cần tạo
BSC_CLUSTER_SIZE=${BSC_CLUSTER_SIZE:-21}  # Default 21 validators
echo "Creating $BSC_CLUSTER_SIZE validator accounts..."

# Tạo validator accounts
for i in $(seq 0 $((BSC_CLUSTER_SIZE - 1))); do
    echo "Creating validator$i..."

    # Tạo directory structure
    mkdir -p "keys/validator$i/keystore"

    # Tạo account mới sử dụng geth
    echo "  - Generating keystore for validator$i..."
    geth account new \
        --keystore "keys/validator$i/keystore" \
        --password "keys/password.txt" \
        > /tmp/geth_output_$i.log 2>&1

    # Lấy địa chỉ được tạo
    if [ -f "/tmp/geth_output_$i.log" ]; then
        ADDRESS=$(grep "Public address of the key" /tmp/geth_output_$i.log | awk '{print $NF}')
        echo "  - Address: $ADDRESS"
        rm "/tmp/geth_output_$i.log"
    fi

    # Tạo node key (64 character hex string)
    echo "  - Generating node key for validator$i..."
    NODE_KEY=$(openssl rand -hex 32)
    echo "$NODE_KEY" > "keys/validator-nodekey$i"

    echo "  ✅ Validator$i created"
done

echo "✅ All validator accounts created"
```

### Bước 4: Tạo BLS Keys (Simplified)

**⚠️ CẢNH BÁO**: Phương pháp này chỉ dành cho development/testnet. Cho mainnet, cần sử dụng tools BLS chuyên dụng.

```bash
echo "Creating BLS keys for $BSC_CLUSTER_SIZE validators..."

for i in $(seq 0 $((BSC_CLUSTER_SIZE - 1))); do
    echo "Creating BLS key for validator$i..."

    # Tạo BLS directory structure
    BLS_DIR="keys/bls$i/bls"
    mkdir -p "$BLS_DIR/keystore"
    mkdir -p "$BLS_DIR/wallet"

    # Generate random BLS public key (96 bytes = 192 hex chars)
    BLS_PUBKEY=$(openssl rand -hex 48)

    # Generate other random values for keystore
    CHECKSUM_MESSAGE=$(openssl rand -hex 32)
    CIPHER_MESSAGE=$(openssl rand -hex 32)
    CIPHER_IV=$(openssl rand -hex 16)
    KDF_SALT=$(openssl rand -hex 32)
    UUID=$(openssl rand -hex 4)-$(openssl rand -hex 2)-$(openssl rand -hex 2)-$(openssl rand -hex 2)-$(openssl rand -hex 6)

    # Create BLS keystore file
    BLS_KEYSTORE_FILE="$BLS_DIR/keystore/keystore-$(openssl rand -hex 8).json"
    cat > "$BLS_KEYSTORE_FILE" << EOF
{
    "crypto": {
        "checksum": {
            "function": "sha256",
            "message": "$CHECKSUM_MESSAGE",
            "params": {}
        },
        "cipher": {
            "function": "aes-128-ctr",
            "message": "$CIPHER_MESSAGE",
            "params": {
                "iv": "$CIPHER_IV"
            }
        },
        "kdf": {
            "function": "pbkdf2",
            "message": "",
            "params": {
                "c": 262144,
                "dklen": 32,
                "prf": "hmac-sha256",
                "salt": "$KDF_SALT"
            }
        }
    },
    "uuid": "$UUID",
    "pubkey": "$BLS_PUBKEY",
    "version": 4,
    "description": "Secure BLS key for validator$i",
    "name": "keystore",
    "path": ""
}
EOF

    echo "  ✅ BLS key for validator$i created"
done

echo "✅ All BLS keys created"
echo "⚠️  WARNING: These are simplified BLS keys for development/testnet."
echo "⚠️  For mainnet, use proper BLS key generation tools!"
```

### Bước 5: Tạo Sentry Node Keys

```bash
echo "Creating sentry node keys for $BSC_CLUSTER_SIZE sentries..."

for i in $(seq 0 $((BSC_CLUSTER_SIZE - 1))); do
    NODEKEY=$(openssl rand -hex 32)
    echo "$nodekey" > "keys/sentry-nodekey$i"
    echo "  ✅ Sentry nodekey$i created"
done

echo "✅ All sentry node keys created"
```

### Bước 6: Tạo Full Node Key

```bash
echo "Creating full node key..."
FULLNODE_KEY=$(openssl rand -hex 32)
echo "$FULLNODE_KEY" > "keys/fullnode-nodekey0"
echo "✅ Full node key created"
```

### Bước 7: Set File Permissions

```bash
echo "Setting secure file permissions..."

# Set restrictive permissions
chmod 600 keys/password.txt
chmod 600 keys/validator-nodekey*
chmod 600 keys/sentry-nodekey*
chmod 600 keys/fullnode-nodekey*
chmod -R 700 keys/validator*/
chmod -R 700 keys/bls*/

# Verify permissions
echo "File permissions set:"
ls -la keys/password.txt
ls -la keys/validator-nodekey* | head -3
ls -ld keys/validator*/ | head -3

echo "✅ File permissions configured"
```

## Phương Pháp Mainnet (Hardware Security)

### Sử dụng Hardware Security Modules (HSM)

Cho mainnet production, cần sử dụng HSM:

```bash
# Ví dụ với AWS CloudHSM
aws cloudhsmv2 create-cluster \
    --backup-retention-policy Type=DAYS,Value=30 \
    --hsm-type hsm1.medium \
    --subnet-ids subnet-12345678

# Ví dụ với YubiHSM
yubihsm-shell --connector http://localhost:12345
```

### Sử dụng Proper BLS Key Generation Tools

```bash
# Ví dụ với eth2-validator-tools
# Cài đặt eth2-validator-tools trước
go install github.com/wealdtech/eth2-validator-tools@latest

# Generate BLS keys
eth2-val-tools keystores \
    --private-key 0x[your-private-key] \
    --password-file keys/password.txt \
    --output-dir keys/bls-secure/

# Ví dụ với Prysm validator tools
# bazel run //validator/accounts/v2:accounts -- wallet create \
#     --wallet-dir keys/prysm-wallet \
#     --wallet-password-file keys/password.txt
```

## Quản Lý Keys An Toàn

### Cập Nhật .gitignore

Đảm bảo private keys không bị commit:

```bash
# Kiểm tra .gitignore hiện tại
if [ -f .gitignore ]; then
    echo "Current .gitignore:"
    cat .gitignore
else
    echo "No .gitignore found, creating one..."
    touch .gitignore
fi

# Thêm rules để ignore keys
cat >> .gitignore << 'EOF'

# Private keys - NEVER commit these!
keys/
.env
*.key
*.keystore
**/keystore/
**/*nodekey*
keys-backup-*/

# Logs may contain sensitive info
*.log
.local/
temp_*
/tmp/geth_output_*.log
EOF

echo "✅ .gitignore updated to exclude private keys"
```

### Verify Git Status

```bash
# Kiểm tra git status để đảm bảo không có keys bị track
echo "Checking git status for keys..."
if git ls-files keys/ 2>/dev/null | grep -q .; then
    echo "❌ WARNING: Keys are tracked in Git!"
    echo "Run: git rm -r --cached keys/"
    echo "Then commit the removal."
else
    echo "✅ Keys are not tracked in Git"
fi
```

### Environment Variables Setup

```bash
# Tạo file .env với paths bảo mật
cat >> .env << EOF

# Security paths (add these to your existing .env)
KEYSTORE_PASSWORD_FILE=$(pwd)/keys/password.txt
VALIDATOR_KEYSTORE_DIR=$(pwd)/keys/
EOF

echo "✅ Environment variables added to .env"
```

## Security Best Practices

### 1. Password Management

```bash
# Test password strength
PASSWORD=$(head -n 1 keys/password.txt)
PASSWORD_LENGTH=${#PASSWORD}

echo "Password analysis:"
echo "  Length: $PASSWORD_LENGTH characters"

if [ $PASSWORD_LENGTH -lt 20 ]; then
    echo "  ❌ Password too short (< 20 chars)"
else
    echo "  ✅ Password length OK"
fi

# Check for common patterns
if echo "$PASSWORD" | grep -q "password\|123456\|qwerty"; then
    echo "  ❌ Password contains common patterns"
else
    echo "  ✅ Password doesn't contain common patterns"
fi
```

### 2. Backup Strategy

Implement 3-2-1 backup rule:

```bash
# Create encrypted backup
BACKUP_DIR="/secure/backup/$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"

echo "Creating encrypted backup..."

# Backup keys với mã hóa GPG
tar -czf - keys/ | gpg -c --batch --passphrase "$PASSWORD" > "$BACKUP_DIR/keys-encrypted.tar.gz.gpg"

# Backup configurations
cp .env "$BACKUP_DIR/" 2>/dev/null || echo "No .env to backup"
cp config.toml "$BACKUP_DIR/"

# Create checksum để verify backup integrity
sha256sum "$BACKUP_DIR"/* > "$BACKUP_DIR/checksums.txt"

echo "✅ Encrypted backup created: $BACKUP_DIR"
echo "📋 Backup includes:"
ls -la "$BACKUP_DIR"
```

### 3. Access Control

```bash
# Create dedicated user cho validator (optional)
echo "Setting up access control..."

# Option 1: Use current user với restricted permissions
sudo chown -R $USER:$USER keys/
sudo chmod -R 700 keys/

# Option 2: Create dedicated validator user (comment out if not needed)
# sudo useradd -m -s /bin/bash validator
# sudo chown -R validator:validator keys/
# sudo chmod -R 700 keys/

echo "✅ Access control configured"
```

### 4. Verification

```bash
# Comprehensive security check
echo "=== BSC Key Security Audit ==="

# Check password strength
PASSWORD=$(head -n 1 keys/password.txt)
if [ "$PASSWORD" = "0123456789" ]; then
    echo "❌ WEAK PASSWORD DETECTED!"
    echo "   Current: $PASSWORD"
    echo "   Action: Run key generation again with stronger password"
else
    echo "✅ Password appears secure"
fi

# Check file permissions
PASS_PERMS=$(stat -c %a keys/password.txt 2>/dev/null || stat -f %A keys/password.txt)
if [ "$PASS_PERMS" != "600" ]; then
    echo "❌ Insecure password file permissions: $PASS_PERMS"
    chmod 600 keys/password.txt
    echo "   Fixed: Set to 600"
else
    echo "✅ Password file permissions OK: $PASS_PERMS"
fi

# Check git tracking
if git ls-files keys/ 2>/dev/null | grep -q .; then
    echo "❌ Keys are tracked in Git!"
    echo "   Action: git rm -r --cached keys/"
else
    echo "✅ Keys not tracked in Git"
fi

# Check if keys exist
VALIDATOR_COUNT=$(ls keys/validator-nodekey* 2>/dev/null | wc -l)
echo "📊 Validator keys created: $VALIDATOR_COUNT"

BLS_COUNT=$(ls keys/bls*/bls/keystore/* 2>/dev/null | wc -l)
echo "📊 BLS keys created: $BLS_COUNT"

echo "=== End Security Audit ==="
```

## Testing Key Security

### Load Test với Password

```bash
# Test load keystore với password mới
echo "Testing keystore loading..."

# Test first validator keystore
FIRST_KEYSTORE=$(ls keys/validator0/keystore/* 2>/dev/null | head -1)
if [ -f "$FIRST_KEYSTORE" ]; then
    echo "Testing keystore: $FIRST_KEYSTORE"

    # Try to get address from keystore
    ADDRESS=$(geth account list --keystore keys/validator0/keystore 2>/dev/null | head -1 | grep -o "0x[a-fA-F0-9]*" | head -1)
    if [ -n "$ADDRESS" ]; then
        echo "✅ Keystore readable, address: $ADDRESS"
    else
        echo "❌ Failed to read keystore"
    fi
else
    echo "❌ No keystore found for validator0"
fi
```

### Network Integration Test

```bash
# Test integration với BSC cluster
echo "Testing network integration..."

# Check if all required files exist
echo "Required files check:"
for i in $(seq 0 $((BSC_CLUSTER_SIZE - 1))); do
    if [ -f "keys/validator-nodekey$i" ] && [ -d "keys/validator$i/keystore" ] && [ -d "keys/bls$i/bls/keystore" ]; then
        echo "  ✅ Validator$i: Complete"
    else
        echo "  ❌ Validator$i: Missing files"
    fi
done
```

## Recovery Procedures

### Khi Mất Password

```bash
# Nếu mất password, keys sẽ KHÔNG THỂ khôi phục
echo "Password Recovery Options:"
echo "1. ✅ Restore từ backup (nếu có)"
echo "2. ❌ Không thể crack password (by design)"
echo "3. ❌ Phải tạo validator set hoàn toàn mới"

# Script để restore từ backup
restore_from_backup() {
    local backup_dir=$1
    if [ -z "$backup_dir" ]; then
        echo "Usage: restore_from_backup <backup_directory>"
        return 1
    fi

    echo "Restoring from backup: $backup_dir"

    # Decrypt và restore
    if [ -f "$backup_dir/keys-encrypted.tar.gz.gpg" ]; then
        echo "Enter decryption password:"
        gpg -d "$backup_dir/keys-encrypted.tar.gz.gpg" | tar -xzf -
        echo "✅ Keys restored from encrypted backup"
    else
        echo "❌ No encrypted backup found"
    fi
}
```

### Emergency Key Rotation

```bash
# Script thay thế keys trong trường hợp khẩn cấp
emergency_key_rotation() {
    echo "🚨 EMERGENCY KEY ROTATION 🚨"
    echo "This will:"
    echo "1. Stop all validators"
    echo "2. Create new keys"
    echo "3. Restart with new keys"

    read -p "Continue? (yes/no): " confirm
    if [ "$confirm" != "yes" ]; then
        echo "Cancelled"
        return 1
    fi

    # Stop cluster
    echo "Stopping cluster..."
    bash bsc_cluster.sh stop

    # Backup old keys
    OLD_KEYS_BACKUP="emergency-backup-$(date +%Y%m%d-%H%M%S)"
    mv keys/ "$OLD_KEYS_BACKUP/"

    # Create new keys (rerun this guide)
    echo "Creating new keys..."
    # ... (repeat key generation steps above)

    # Restart cluster
    echo "Restarting cluster with new keys..."
    bash bsc_cluster.sh reset

    echo "✅ Emergency rotation complete"
    echo "📦 Old keys backed up to: $OLD_KEYS_BACKUP"
}
```

## Production Mainnet Considerations

### Compliance Requirements

```bash
# Document key generation process
cat > "key-generation-log-$(date +%Y%m%d).md" << 'EOF'
# BSC Key Generation Log

## Date
Generated on: $(date)

## Method
- Manual key generation using OpenSSL
- Secure random number generation
- AES-256 keystore encryption

## Security Measures
- [ ] Secure password (25+ characters)
- [ ] Restricted file permissions (600/700)
- [ ] Encrypted backups
- [ ] Air-gapped generation environment
- [ ] HSM integration (mainnet only)

## Validation
- [ ] Keys load successfully
- [ ] No keys in version control
- [ ] Backup tested and verified
- [ ] Security audit completed

## Notes
Add any specific notes about the generation process...
EOF

echo "✅ Key generation documented"
```

### Monitoring Setup

```bash
# Setup key file monitoring
setup_key_monitoring() {
    echo "Setting up key file monitoring..."

    # Monitor key file access (Linux with auditd)
    if command -v auditctl >/dev/null 2>&1; then
        sudo auditctl -w $(pwd)/keys/ -p r -k bsc_key_access
        echo "✅ Audit monitoring enabled"
    else
        echo "⚠️  auditd not available, consider installing for monitoring"
    fi

    # Simple file integrity monitoring
    find keys/ -type f -exec sha256sum {} \; > keys-integrity.sha256
    echo "✅ Integrity baseline created: keys-integrity.sha256"
}
```

## Tài Liệu Tham Khảo

### Security Standards

- [NIST Cryptographic Standards](https://csrc.nist.gov/projects/cryptographic-standards-and-guidelines)
- [OWASP Key Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Key_Management_Cheat_Sheet.html)
- [BSC Security Best Practices](https://docs.bnbchain.org/docs/validator/security)

### Tools

- [Ethereum Keystore Utils](https://github.com/ethereum/eth-keyfile)
- [BLS Signature Tools](https://github.com/supranational/blst)
- [Hardware Security Modules](https://aws.amazon.com/cloudhsm/)

### Related Guidelines

- [Switch Network Type](./1-switch-network-type.md)
- [Change Chain ID](./0-change-chainid.md)

---

## 📝 Final Checklist

Sau khi hoàn thành tất cả các bước:

- [ ] ✅ Password mạnh đã được tạo và lưu an toàn
- [ ] ✅ Validator accounts đã được tạo cho tất cả validators
- [ ] ✅ BLS keys đã được tạo (simplified cho dev/testnet)
- [ ] ✅ Node keys đã được tạo
- [ ] ✅ File permissions đã được set đúng (600/700)
- [ ] ✅ Keys không bị track trong Git
- [ ] ✅ Backup được tạo và test
- [ ] ✅ Security audit đã chạy và pass
- [ ] ✅ Documentation đã được tạo

🎉 **Keys đã sẵn sàng để sử dụng!**
