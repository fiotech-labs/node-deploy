# 1. Hướng Dẫn Chuyển Đổi Loại Blockchain (Development ↔ Testnet ↔ Mainnet)

## Tổng Quan

Hướng dẫn này sẽ giúp bạn chuyển đổi giữa các loại blockchain BSC:

- **Development**: Blockchain phát triển local (Chain ID: 19999)
- **Testnet**: BSC Testnet chính thức (Chain ID: 97)
- **Mainnet**: BSC Mainnet chính thức (Chain ID: 56)

## Phân Biệt Các Loại Blockchain

### Development (Hiện tại)

- **Chain ID**: 19999
- **Mục đích**: Phát triển và testing local
- **Số validators**: 21 (có thể tùy chỉnh)
- **Block time**: 3 giây
- **Hardfork delays**: Có delay để testing
- **Keys**: Sử dụng keys mẫu không bảo mật

### Testnet

- **Chain ID**: 97
- **Mục đích**: Testing trên mạng public trước khi deploy mainnet
- **Số validators**: 6-21 validators
- **Block time**: 3 giây
- **Hardfork delays**: Không có delay
- **Keys**: Cần tạo keys bảo mật riêng

### Mainnet

- **Chain ID**: 56
- **Mục đích**: Production network chính thức
- **Số validators**: 45+ validators
- **Block time**: 3 giây
- **Hardfork delays**: Không có delay
- **Keys**: Cần tạo keys bảo mật cao nhất

## Cách Chuyển Đổi (Thủ Công)

### Bước 1: Tạo File Environment (.env)

Tạo file `.env` trong thư mục gốc của project:

#### Cho Development:

```bash
# Tạo file .env cho development
cat > .env << 'EOF'
BSC_CLUSTER_SIZE=21
CHAIN_ID=19999
KEYPASS="0123456789"
INIT_HOLDER="0x04d63aBCd2b9b1baa327f2Dda0f873F197ccd186"
# INIT_HOLDER_PRV="59ba8068eb256d520179e903f43dacf6d8d57d72bd306e1bd603fdb8c8da10e8"
RPC_URL="http://127.0.0.1:8545"
GENESIS_COMMIT="4e0cc71423adddcae475ed3c3819d9816a7d0245" # lorentz commit
PASSED_FORK_DELAY=20
LAST_FORK_MORE_DELAY=10
FullImmutabilityThreshold=1000000
MinBlocksForBlobRequests=32
DefaultExtraReserveForBlobRequests=32
BreatheBlockInterval=600
useLatestBscClient=false
EnableSentryNode=false
EnableFullNode=false
RegisterNodeID=false
EnableEVNWhitelist=false
EOF
```

#### Cho Testnet:

```bash
# Tạo file .env cho testnet
cat > .env << 'EOF'
BSC_CLUSTER_SIZE=6
CHAIN_ID=97
KEYPASS="[YOUR_SECURE_PASSWORD]"     # ⚠️ THAY ĐỔI PASSWORD!
INIT_HOLDER="[YOUR_SECURE_TESTNET_ADDRESS]"  # ⚠️ THAY ĐỔI ADDRESS!
# INIT_HOLDER_PRV="[YOUR_PRIVATE_KEY]"
RPC_URL="http://127.0.0.1:8545"
GENESIS_COMMIT="4e0cc71423adddcae475ed3c3819d9816a7d0245" # lorentz commit
PASSED_FORK_DELAY=0
LAST_FORK_MORE_DELAY=0
FullImmutabilityThreshold=1000000
MinBlocksForBlobRequests=576
DefaultExtraReserveForBlobRequests=32
BreatheBlockInterval=86400
useLatestBscClient=false
EnableSentryNode=false
EnableFullNode=false
RegisterNodeID=false
EnableEVNWhitelist=false
EOF
```

#### Cho Mainnet:

```bash
# Tạo file .env cho mainnet
cat > .env << 'EOF'
BSC_CLUSTER_SIZE=45
CHAIN_ID=56
KEYPASS="[YOUR_HSM_SECURED_PASSWORD]"  # ⚠️ PHẢI DÙNG HSM PASSWORD!
INIT_HOLDER="[YOUR_HSM_SECURED_ADDRESS]"  # ⚠️ PHẢI DÙNG HSM ADDRESS!
# INIT_HOLDER_PRV="[YOUR_HSM_PRIVATE_KEY]"
RPC_URL="http://127.0.0.1:8545"
GENESIS_COMMIT="4e0cc71423adddcae475ed3c3819d9816a7d0245" # lorentz commit
PASSED_FORK_DELAY=0
LAST_FORK_MORE_DELAY=0
FullImmutabilityThreshold=1000000
MinBlocksForBlobRequests=576
DefaultExtraReserveForBlobRequests=32
BreatheBlockInterval=86400
useLatestBscClient=false
EnableSentryNode=true
EnableFullNode=false
RegisterNodeID=false
EnableEVNWhitelist=true
EOF
```

### Bước 2: Cập Nhật config.toml

Chỉnh sửa `NetworkId` trong file `config.toml`:

```bash
# Cho Development (Chain ID 19999)
sed -i.bak "s/NetworkId = .*/NetworkId = 19999/" config.toml

# Cho Testnet (Chain ID 97)
sed -i.bak "s/NetworkId = .*/NetworkId = 97/" config.toml

# Cho Mainnet (Chain ID 56)
sed -i.bak "s/NetworkId = .*/NetworkId = 56/" config.toml
```

### Bước 3: Generate Genesis Configuration

Di chuyển vào thư mục genesis và tạo cấu hình genesis:

```bash
cd genesis

# Cài đặt dependencies nếu chưa có
if [ ! -d "node_modules" ]; then
    echo "Installing genesis dependencies..."
    npm install
fi

# Cài đặt Python dependencies
poetry install --no-root

# Generate genesis theo network type
```

#### Cho Development:

```bash
poetry run python -m scripts.generate dev \
    --dev-chain-id 19999 \
    --init-burn-ratio "1000" \
    --init-felony-slash-scope "60" \
    --breathe-block-interval "10 minutes" \
    --block-interval "3 seconds" \
    --stake-hub-protector "0x08E68Ec70FA3b629784fDB28887e206ce8561E08" \
    --unbond-period "2 minutes" \
    --downtime-jail-time "2 minutes" \
    --felony-jail-time "3 minutes" \
    --misdemeanor-threshold "50" \
    --felony-threshold "150" \
    --init-voting-period "2 minutes / BLOCK_INTERVAL" \
    --init-min-period-after-quorum "uint64(1 minutes / BLOCK_INTERVAL)" \
    --governor-protector "0x08E68Ec70FA3b629784fDB28887e206ce8561E08" \
    --init-minimal-delay "1 minutes" \
    --token-recover-portal-protector "0x08E68Ec70FA3b629784fDB28887e206ce8561E08"
```

#### Cho Testnet:

```bash
poetry run python -m scripts.generate testnet
```

#### Cho Mainnet:

```bash
poetry run python -m scripts.generate mainnet
```

### Bước 4: Clean Up và Khởi Động

```bash
# Quay về thư mục gốc
cd ..

# Xóa dữ liệu cũ
rm -rf .local/
rm -f *.log

# Khởi động blockchain với cấu hình mới
bash -x ./bsc_cluster.sh reset
```

## Cấu Hình Chi Tiết

### Development Parameters

```bash
# Network settings
CHAIN_ID=19999                    # Development chain ID
BSC_CLUSTER_SIZE=21              # Number of validators
PASSED_FORK_DELAY=20             # Fork delay for testing
LAST_FORK_MORE_DELAY=10          # Additional delay

# Performance settings
FullImmutabilityThreshold=1000000 # Immutability threshold
BreatheBlockInterval=600          # Breathing interval (10 minutes)
MinBlocksForBlobRequests=32       # Min blocks for blob
DefaultExtraReserveForBlobRequests=32 # Extra reserve

# Initial holder (can use development address)
INIT_HOLDER="0x04d63aBCd2b9b1baa327f2Dda0f873F197ccd186"
```

### Testnet Parameters

```bash
# Network settings
CHAIN_ID=97                      # BSC testnet chain ID
BSC_CLUSTER_SIZE=6               # Fewer validators for testnet
PASSED_FORK_DELAY=0              # No delay for testnet
LAST_FORK_MORE_DELAY=0           # No additional delay

# Performance settings
BreatheBlockInterval=86400        # 24 hours breathing interval

# Security: Must use secure address for testnet
INIT_HOLDER="0x[YOUR_SECURE_ADDRESS]"
```

### Mainnet Parameters

```bash
# Network settings
CHAIN_ID=56                      # BSC mainnet chain ID
BSC_CLUSTER_SIZE=45              # Full validator set
PASSED_FORK_DELAY=0              # No delay for production
LAST_FORK_MORE_DELAY=0           # No additional delay

# Performance settings
BreatheBlockInterval=86400        # 24 hours breathing interval

# Security: Must use secure address for mainnet
INIT_HOLDER="0x[YOUR_SECURE_ADDRESS]"
```

## Kiểm Tra Kết Quả

### Verification Commands

```bash
# 1. Kiểm tra Chain ID trong config
echo "Config.toml NetworkId:"
grep "NetworkId" config.toml

# 2. Kiểm tra environment variables
echo "Environment CHAIN_ID:"
grep "CHAIN_ID" .env

# 3. Kiểm tra genesis file (sau khi generate)
echo "Genesis chain ID:"
cat genesis/genesis.json | jq .config.chainId

# 4. Kiểm tra file .env đã tạo
echo "Environment file contents:"
cat .env

# 5. Verify BSC source code (nếu đã modify)
echo "BSC source code RialtoChainConfig:"
grep -A 5 "RialtoChainConfig" bsc/params/config.go || echo "BSC source not modified"
```

### Post-Start Verification

Sau khi khởi động blockchain bằng `bsc_cluster.sh reset`, kiểm tra:

```bash
# Đợi network khởi động
sleep 30

# Kiểm tra chain ID via RPC
echo "Checking chain ID via RPC..."
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_chainId","params":[],"id":1}' \
  -H "Content-Type: application/json" http://localhost:8545

# Kiểm tra block number
echo "Checking latest block..."
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  -H "Content-Type: application/json" http://localhost:8545

# Kiểm tra network ID
echo "Checking network ID..."
curl -X POST --data '{"jsonrpc":"2.0","method":"net_version","params":[],"id":1}' \
  -H "Content-Type: application/json" http://localhost:8545
```

### Expected Results:

- **Development**: chainId `"0x4e1f"` (19999)
- **Testnet**: chainId `"0x61"` (97)
- **Mainnet**: chainId `"0x38"` (56)

## ⚠️ Cảnh Báo Bảo Mật

### Khi chuyển sang Testnet:

- [ ] **BẮT BUỘC**: Tạo keys mới theo [Generate Secure Keys](./2-generate-secure-keys.md)
- [ ] **BẮT BUỘC**: Thay đổi `INIT_HOLDER` thành địa chỉ của bạn
- [ ] **BẮT BUỘC**: Sử dụng password mạnh và unique
- [ ] **BẮT BUỘC**: Không commit private keys vào git
- [ ] **KHUYẾN NGHỊ**: Backup keys an toàn

### Khi chuyển sang Mainnet:

- [ ] **BẮT BUỘC**: Tất cả yêu cầu của Testnet
- [ ] **BẮT BUỘC**: Thay đổi `INIT_HOLDER` thành địa chỉ bảo mật cao
- [ ] **BẮT BUỘC**: Sử dụng Hardware Security Modules (HSM)
- [ ] **BẮT BUỘC**: Professional security audit
- [ ] **BẮT BUỘC**: Multi-location secure backup
- [ ] **BẮT BUỘC**: Proper BLS key generation tools
- [ ] **BẮT BUỘC**: Insurance coverage

## 🔧 Troubleshooting

### Lỗi "Poetry not found"

```bash
# Cài đặt poetry
curl -sSL https://install.python-poetry.org | python3 -

# Hoặc với pip
pip3 install poetry

# Reload shell
source ~/.bashrc  # hoặc ~/.zshrc
```

### Lỗi "npm dependencies missing"

```bash
cd genesis
npm install
```

### Lỗi genesis generation

```bash
cd genesis

# Reinstall Python dependencies
poetry install --no-root

# Check available commands
poetry run python -m scripts.generate --help

# Debug specific errors
poetry run python -m scripts.generate dev --help
```

### Lỗi Chain ID không đổi (vẫn là 714)

```bash
# Kiểm tra BSC source code đã được modify chưa
grep -n "714" bsc/params/config.go

# Nếu chưa modify, cần thay đổi hardcoded value:
# Xem hướng dẫn trong file 0-change-chainid.md

# Rebuild BSC sau khi modify
cd bsc && make geth && cp build/bin/geth ../bin/geth
```

### Clean up khi có lỗi

```bash
# Xóa dữ liệu cũ
rm -rf .local/
rm -f *.log

# Reset genesis
cd genesis
git reset --hard HEAD
cd ..

# Tạo lại từ đầu
# Repeat các bước trên
```

## Backup và Recovery

### Backup trước khi chuyển đổi:

```bash
# Backup toàn bộ cấu hình hiện tại
BACKUP_DIR="backup-$(date +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP_DIR"

# Backup keys (nếu có)
if [ -d keys/ ]; then
    cp -r keys/ "$BACKUP_DIR/"
fi

# Backup configs
cp .env "$BACKUP_DIR/" 2>/dev/null || echo "No .env to backup"
cp config.toml "$BACKUP_DIR/"

# Backup genesis config
cp -r genesis/ "$BACKUP_DIR/"

echo "Backup saved to: $BACKUP_DIR"
```

### Recovery nếu có vấn đề:

```bash
# List available backups
ls -la backup-*/

# Restore từ backup (replace TIMESTAMP)
BACKUP_DIR="backup-YYYYMMDD-HHMMSS"

# Restore keys
if [ -d "$BACKUP_DIR/keys/" ]; then
    cp -r "$BACKUP_DIR/keys/" ./
fi

# Restore configs
cp "$BACKUP_DIR/.env" ./ 2>/dev/null || echo "No .env in backup"
cp "$BACKUP_DIR/config.toml" ./

# Restore genesis
cp -r "$BACKUP_DIR/genesis/" ./

# Clean và restart
rm -rf .local/
bash -x ./bsc_cluster.sh reset
```

### Emergency Reset về Development:

```bash
# Quick reset về development setup
cat > .env << 'EOF'
BSC_CLUSTER_SIZE=21
CHAIN_ID=19999
KEYPASS="0123456789"
INIT_HOLDER="0x04d63aBCd2b9b1baa327f2Dda0f873F197ccd186"
# INIT_HOLDER_PRV="59ba8068eb256d520179e903f43dacf6d8d57d72bd306e1bd603fdb8c8da10e8"
RPC_URL="http://127.0.0.1:8545"
GENESIS_COMMIT="4e0cc71423adddcae475ed3c3819d9816a7d0245" # lorentz commit
PASSED_FORK_DELAY=20
LAST_FORK_MORE_DELAY=10
FullImmutabilityThreshold=1000000
MinBlocksForBlobRequests=32
DefaultExtraReserveForBlobRequests=32
BreatheBlockInterval=600
useLatestBscClient=false
EnableSentryNode=false
EnableFullNode=false
RegisterNodeID=false
EnableEVNWhitelist=false
EOF

# Update config.toml
sed -i.bak "s/NetworkId = .*/NetworkId = 19999/" config.toml

# Reset genesis và restart
cd genesis
poetry run python -m scripts.generate dev --dev-chain-id 19999
cd ..
rm -rf .local/
bash -x ./bsc_cluster.sh reset
```

## Tham Khảo

- [BSC Official Documentation](https://docs.bnbchain.org/)
- [BSC Testnet Info](https://testnet.bscscan.com/)
- [BSC Mainnet Info](https://bscscan.com/)
- [Guidelines: Generate Secure Keys](./2-generate-secure-keys.md)
- [Guidelines: Change Chain ID](./0-change-chainid.md)
