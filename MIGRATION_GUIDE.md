# Hướng dẫn Migration Database và Cấu hình .env

## 1. Migration Database vào PostgreSQL (Supabase)

### Cách 1: Sử dụng Supabase Dashboard (Khuyến nghị)

1. **Tạo project Supabase:**
   - Truy cập https://supabase.com
   - Tạo project mới
   - Lưu lại các thông tin:
     - `SUPABASE_URL` (Project URL)
     - `SUPABASE_SERVICE_ROLE_KEY` (Settings > API > service_role key)
     - `DATABASE_POOLER_URL` (Settings > Database > Connection string > Transaction mode)

2. **Chạy migration SQL:**
   - Vào SQL Editor trong Supabase Dashboard
   - Chạy file migration theo thứ tự:
     - `backend/supabase/migrations/20250102000001_initial_schema.sql` (tạo tables, functions, triggers)
     - `backend/supabase/migrations/20250102000002_cron_jobs.sql` (thiết lập scheduled jobs)
     - `backend/supabase/migrations/20250102000003_author_wallets.sql` (tạo author_wallets table cho dual-wallet support)
     - `backend/supabase/migrations/20250102000004_add_missing_columns.sql` (thêm các columns còn thiếu: drafts.is_auto_save, articles.author_primary_network, etc.)

3. **Kiểm tra migration:**
   - Vào Table Editor, kiểm tra các tables đã được tạo:
     - `authors`
     - `articles`
     - `user_likes`
     - `drafts`
     - `payments`
     - `author_wallets` (nếu có)

### Cách 2: Sử dụng Supabase CLI (Nâng cao)

```bash
# Cài đặt Supabase CLI
npm install -g supabase

# Login vào Supabase
supabase login

# Link project
supabase link --project-ref <your-project-ref>

# Chạy migrations
supabase db push
```

### Cách 3: Chạy SQL trực tiếp qua psql hoặc pgAdmin

Nếu bạn có PostgreSQL riêng (không dùng Supabase):

```bash
# Kết nối vào database
psql -h <host> -U <user> -d <database>

# Chạy migration files
\i backend/supabase/migrations/20250102000001_initial_schema.sql
\i backend/supabase/migrations/20250102000002_cron_jobs.sql
```

### Import dữ liệu từ SQLite (Nếu có)

Nếu bạn đang migrate từ SQLite sang PostgreSQL:

```bash
# 1. Export dữ liệu từ SQLite
ts-node backend/scripts/exportSQLiteData.ts

# 2. Import vào PostgreSQL
ts-node backend/scripts/importToPostgres.ts
```

**Lưu ý:** Scripts này yêu cầu file `.env` đã được cấu hình với Supabase credentials.

---

## 2. Tạo file .env

### Backend `.env` (tạo trong thư mục `backend/`)

```env
# ============================================
# SUPABASE DATABASE CONFIGURATION
# ============================================
# Bắt buộc - Lấy từ Supabase Dashboard > Settings > API
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key-here

# Bắt buộc - Lấy từ Supabase Dashboard > Settings > Database > Connection string > Transaction mode
DATABASE_POOLER_URL=postgresql://postgres:[YOUR-PASSWORD]@db.your-project.supabase.co:6543/postgres

# ============================================
# SERVER CONFIGURATION
# ============================================
PORT=3001
NODE_ENV=development

# ============================================
# X402 PAYMENT PROTOCOL
# ============================================
# Network: 'base', 'base-sepolia', 'solana', hoặc 'solana-devnet'
X402_NETWORK=base-sepolia

# EVM USDC Addresses (Optional - có giá trị mặc định)
X402_MAINNET_USDC_ADDRESS=0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913
X402_TESTNET_USDC_ADDRESS=0x036CbD53842c5426634e7929541eC2318f3dCF7e

# Solana USDC Mint Addresses (Optional)
X402_SOLANA_MAINNET_USDC_ADDRESS=EPjFWdd5AufqSSqeM2qN1xzybapC8G4wEGGkZwyTDt1v
X402_SOLANA_DEVNET_USDC_ADDRESS=4zMMC9sr...

# Platform Fee Wallets (Optional - có giá trị mặc định)
X402_PLATFORM_EVM_ADDRESS=0x6945890B1c074414b813C7643aE10117dec1C8e7
X402_PLATFORM_SOL_ADDRESS=your-solana-platform-wallet

# Facilitator URL (Optional - mặc định: https://x402.org/facilitator)
X402_FACILITATOR_URL=https://x402.org/facilitator

# ============================================
# COINBASE CDP (Optional - chỉ cần nếu dùng Coinbase settlement)
# ============================================
CDP_API_KEY_ID=your-cdp-api-key-id
CDP_API_KEY_SECRET=your-cdp-api-key-secret

# ============================================
# SOLANA RPC URLs (Optional - có giá trị mặc định)
# ============================================
SOLANA_MAINNET_RPC_URL=https://api.mainnet-beta.solana.com
SOLANA_DEVNET_RPC_URL=https://api.devnet.solana.com
```

### Frontend `.env` (tạo trong thư mục `frontend/`)

```env
# ============================================
# API CONFIGURATION
# ============================================
# URL của backend API
VITE_API_URL=http://localhost:3001/api
# Hoặc dùng VITE_API_BASE_URL (có thể bỏ qua /api)
VITE_API_BASE_URL=http://localhost:3001/api

# ============================================
# WALLET CONNECT
# ============================================
# Bắt buộc - Lấy từ https://cloud.walletconnect.com
VITE_WALLETCONNECT_PROJECT_ID=your-walletconnect-project-id

# ============================================
# X402 PAYMENT PROTOCOL
# ============================================
# Network phải khớp với backend X402_NETWORK
VITE_X402_NETWORK=base-sepolia

# Facilitator URL (Optional - mặc định: https://x402.org/facilitator)
VITE_X402_FACILITATOR_URL=https://x402.org/facilitator

# ============================================
# SOLANA RPC URLs (Optional)
# ============================================
VITE_SOLANA_MAINNET_RPC_URL=https://api.mainnet-beta.solana.com
VITE_SOLANA_DEVNET_RPC_URL=https://api.devnet.solana.com

# ============================================
# COINBASE CDP (Optional)
# ============================================
VITE_COINBASE_CDP_APP_ID=your-coinbase-cdp-app-id
```

---

## 3. Tóm tắt các biến môi trường

### Backend - Bắt buộc:
- ✅ `SUPABASE_URL` - URL của Supabase project
- ✅ `SUPABASE_SERVICE_ROLE_KEY` - Service role key từ Supabase
- ✅ `DATABASE_POOLER_URL` - Connection string cho PostgreSQL pooler

### Backend - Tùy chọn (có giá trị mặc định):
- `PORT` - Port server (mặc định: 3001)
- `X402_NETWORK` - Network cho x402 (mặc định: base - Base Chain mainnet)
- `X402_MAINNET_USDC_ADDRESS` - EVM mainnet USDC address
- `X402_TESTNET_USDC_ADDRESS` - EVM testnet USDC address
- `X402_SOLANA_MAINNET_USDC_ADDRESS` - Solana mainnet USDC mint
- `X402_SOLANA_DEVNET_USDC_ADDRESS` - Solana devnet USDC mint
- `X402_PLATFORM_EVM_ADDRESS` - Platform fee wallet (EVM)
- `X402_PLATFORM_SOL_ADDRESS` - Platform fee wallet (Solana)
- `X402_FACILITATOR_URL` - x402 facilitator URL
- `CDP_API_KEY_ID` - Coinbase CDP API key ID
- `CDP_API_KEY_SECRET` - Coinbase CDP API key secret
- `SOLANA_MAINNET_RPC_URL` - Solana mainnet RPC
- `SOLANA_DEVNET_RPC_URL` - Solana devnet RPC

### Frontend - Bắt buộc:
- ✅ `VITE_WALLETCONNECT_PROJECT_ID` - WalletConnect project ID

### Frontend - Tùy chọn (có giá trị mặc định):
- `VITE_API_URL` hoặc `VITE_API_BASE_URL` - Backend API URL (mặc định: http://localhost:3001/api)
- `VITE_X402_NETWORK` - Network cho x402 (nên khớp với backend)
- `VITE_X402_FACILITATOR_URL` - x402 facilitator URL
- `VITE_SOLANA_MAINNET_RPC_URL` - Solana mainnet RPC
- `VITE_SOLANA_DEVNET_RPC_URL` - Solana devnet RPC
- `VITE_COINBASE_CDP_APP_ID` - Coinbase CDP app ID

---

## 4. Kiểm tra sau khi setup

### Kiểm tra Backend:
```bash
cd backend
npm run dev
```

Truy cập: http://localhost:3001/api/health

Kết quả mong đợi:
```json
{
  "message": "WritingAndEarn.xyz backend is running!",
  "timestamp": "...",
  "version": "1.0.0",
  "facilitator": "...",
  "network": "base-sepolia",
  "cdpEnabled": false
}
```

### Kiểm tra Frontend:
```bash
cd frontend
npm run dev
```

Truy cập: http://localhost:3000

### Kiểm tra Database Connection:
Backend sẽ tự động test connection khi khởi động. Nếu thấy log:
```
✅ Connected to Supabase PostgreSQL
✅ PostgreSQL pool connected to Supabase
✅ Database connection test successful
```

Thì database đã được kết nối thành công!

---

## 5. Troubleshooting

### Lỗi: "Missing required environment variables"
- Kiểm tra file `.env` đã được tạo đúng thư mục (`backend/.env` hoặc `frontend/.env`)
- Kiểm tra tên biến đã đúng (case-sensitive)
- Đảm bảo không có khoảng trắng thừa trong giá trị

### Lỗi: "Database connection failed"
- Kiểm tra `SUPABASE_URL` và `SUPABASE_SERVICE_ROLE_KEY` đã đúng
- Kiểm tra `DATABASE_POOLER_URL` đã đúng format và password
- Kiểm tra firewall/network có chặn kết nối không

### Lỗi: "Table does not exist"
- Chạy lại migration SQL trong Supabase Dashboard
- Kiểm tra migration files đã được chạy đúng thứ tự

### Lỗi: "WalletConnect project ID required"
- Đăng ký project tại https://cloud.walletconnect.com
- Lấy Project ID và thêm vào `VITE_WALLETCONNECT_PROJECT_ID`

---

## 6. Tài liệu tham khảo

- Supabase Dashboard: https://app.supabase.com
- WalletConnect Cloud: https://cloud.walletconnect.com
- x402 Protocol: https://x402.org
- Migration files: `backend/supabase/migrations/`

