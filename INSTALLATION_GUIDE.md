# 📦 Panduan Instalasi Lengkap - Proctoring Application

Dokumen ini berisi panduan step-by-step untuk menginstal dan menjalankan Proctoring Application di berbagai platform.

---

## 🚀 Quick Start (5 Menit)

Jika Anda hanya ingin menjalankan cepat:

```bash
# 1. Clone repository
git clone https://github.com/yourusername/proctoring-app.git
cd proctoring-app

# 2. Setup virtual environment
python -m venv venv
source venv/bin/activate  # macOS/Linux
# atau
venv\Scripts\activate     # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Setup .env
cp .env.example .env
# Edit .env dengan kredensial database Anda

# 5. Create database
mysql -u root -p < DATABASE_MIGRATION.sql

# 6. Run aplikasi
python app.py

# 7. Akses di browser
# Admin: http://localhost:5000/admin/login
# Peserta: http://localhost:5000/login
```

---

## 📋 Prerequisites Checklist

Sebelum memulai, pastikan Anda sudah memiliki:

- [ ] Python 3.10+ terinstall
- [ ] MySQL Server running
- [ ] Git terinstall
- [ ] Text editor (VS Code recommended)
- [ ] Internet connection (untuk download dependencies)

---

## 🔧 Instalasi Detail

### Step 1: Persiapan Sistem

#### Windows

**A. Install Python**
1. Download dari [python.org](https://www.python.org/downloads/)
2. Pilih Python 3.10+ 
3. **PENTING**: Centang "Add Python to PATH"
4. Click "Install Now"

```bash
# Verifikasi instalasi
python --version
pip --version
```

**B. Install MySQL**
1. Download dari [mysql.com](https://dev.mysql.com/downloads/mysql/)
2. Jalankan installer
3. Choose "Development Default" setup
4. Ikuti wizard sampai selesai

```bash
# Verifikasi instalasi
mysql --version
mysql -u root -p  # Test login
```

**C. Install Git**
1. Download dari [git-scm.com](https://git-scm.com)
2. Jalankan installer
3. Gunakan default settings

```bash
# Verifikasi instalasi
git --version
```

---

#### macOS

**Menggunakan Homebrew (Recommended)**
```bash
# Install Homebrew (jika belum punya)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install dependencies
brew install python@3.10 mysql git

# Verifikasi
python3 --version
mysql --version
git --version
```

**Atau manual:**
1. Download Python dari [python.org](https://www.python.org/downloads/)
2. Download MySQL dari [mysql.com](https://dev.mysql.com/downloads/mysql/)
3. Download Git dari [git-scm.com](https://git-scm.com)

---

#### Linux (Ubuntu/Debian)

```bash
# Update package manager
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install python3.10 python3-pip mysql-server git -y

# Start MySQL service
sudo systemctl start mysql
sudo systemctl enable mysql

# Verifikasi
python3 --version
mysql --version
git --version
```

---

### Step 2: Clone Repository

```bash
# Navigate ke folder project
cd ~/Documents  # atau folder pilihan Anda

# Clone repository
git clone https://github.com/yourusername/proctoring-app.git

# Masuk ke directory project
cd proctoring-app

# List files
ls -la
```

---

### Step 3: Setup Virtual Environment

#### Windows

```bash
# Buat virtual environment
python -m venv proctoring_env

# Aktifkan
proctoring_env\Scripts\activate

# Verifikasi (prompt akan berubah)
# (proctoring_env) C:\Users\...>
```

#### macOS/Linux

```bash
# Buat virtual environment
python3 -m venv proctoring_env

# Aktifkan
source proctoring_env/bin/activate

# Verifikasi (prompt akan berubah)
# (proctoring_env) $
```

---

### Step 4: Install Python Dependencies

```bash
# Pastikan virtual environment aktif
# Prompt harus menunjukkan (proctoring_env)

# Upgrade pip dulu (optional tapi recommended)
pip install --upgrade pip setuptools wheel

# Install requirements
pip install -r requirements.txt

# Proses ini akan memakan waktu 5-15 menit tergantung koneksi
# Ukuran download: ~1.2 GB

# Verifikasi instalasi
pip list
```

**Jika ada error saat install:**

```bash
# Error: "Could not find a version that satisfies the requirement"
# → Cek internet connection
# → Coba ulang: pip install -r requirements.txt

# Error: "Microsoft Visual C++ 14.0 is required" (Windows)
# → Download Visual C++ Build Tools
# → https://visualstudio.microsoft.com/visual-cpp-build-tools/
# → Install dan restart

# Error: dlib installation failed
# → Windows: Install dari file wheel
#   pip install dlib-19.24.4-cp310-cp310-win_amd64.whl
# → macOS: brew install dlib
# → Linux: sudo apt install libopenblas-dev
```

---

### Step 5: Setup Environment Variables

```bash
# Copy template ke .env
cp .env.example .env

# Buka file .env dengan text editor (atau command di bawah)
# Windows
notepad .env

# macOS/Linux
nano .env
```

**Edit .env dengan konfigurasi Anda:**

```ini
# Server
SERVER_HOST=localhost
PORT=5000
SECRET_KEY=your_secure_key_12345

# Database (sesuaikan)
DB_HOST=localhost
DB_PORT=3306
DB_NAME=proctoring_app
DB_USER=root
DB_PASSWORD=your_mysql_password

# Admin
ADMIN_USERNAME=admin
ADMIN_PASSWORD=admin123

# Cloudinary (optional)
CLOUDINARY_CLOUD_NAME=your_name
CLOUDINARY_API_KEY=your_key
CLOUDINARY_API_SECRET=your_secret
```

---

### Step 6: Setup Database

#### Option A: Command Line

```bash
# Login ke MySQL
mysql -u root -p

# Dalam MySQL prompt:
CREATE DATABASE proctoring_app;
USE proctoring_app;

# Exit
exit;

# Run migration script
mysql -u root -p proctoring_app < DATABASE_MIGRATION.sql

# Verifikasi
mysql -u root -p proctoring_app -e "SHOW TABLES;"
```

#### Option B: MySQL Workbench (GUI)

1. Buka MySQL Workbench
2. Buat koneksi lokal
3. Create new schema: `proctoring_app`
4. Run SQL script: `DATABASE_MIGRATION.sql`

#### Option C: DBeaver (GUI)

1. Buka DBeaver
2. Koneksikan ke MySQL server lokal
3. Create database `proctoring_app`
4. Execute `DATABASE_MIGRATION.sql`

---

### Step 7: Verifikasi Setup

```bash
# Pastikan virtual environment masih aktif

# Test database connection
python check_current_state.py

# Test exam URLs
python check_exam_urls.py

# Output yang diharapkan:
# ✅ Database connected
# ✅ Tables created
# ✅ Sample data ready
```

---

## ▶️ Menjalankan Aplikasi

### Development Mode

```bash
# Pastikan virtual environment aktif
source proctoring_env/bin/activate  # macOS/Linux
# atau
proctoring_env\Scripts\activate     # Windows

# Run Flask
python app.py

# Output:
# * Running on http://localhost:5000
# * Debug mode: on
# * WARNING: This is a development server. Do not use it in production.
```

### Akses Aplikasi

Buka browser dan go to:

- **Admin Panel**: http://localhost:5000/admin/login
  - Username: `admin`
  - Password: (sesuai .env)

- **Peserta Portal**: http://localhost:5000/login

---

## 🧪 Testing

Sebelum production, jalankan tests:

```bash
# Test 1: System health check
python test_system.py

# Test 2: Database connectivity
python verify_structure.py

# Test 3: Violation API
python test_violation_api_fixed.py

# Test 4: Violations table
python test_violations_table.py
```

---

## 📊 Database Verification

```bash
# Connect ke database
mysql -u root -p proctoring_app

# Di MySQL prompt, jalankan:
SHOW TABLES;
DESCRIBE sessions;
DESCRIBE participants;
DESCRIBE violations;
DESCRIBE violation_evidence;

# Exit
exit;
```

---

## 🚨 Common Issues & Solutions

### Issue: "ModuleNotFoundError: No module named 'flask'"

**Penyebab**: Virtual environment tidak aktif

**Solusi**:
```bash
# Aktifkan virtual environment
source proctoring_env/bin/activate  # macOS/Linux
proctoring_env\Scripts\activate     # Windows
```

---

### Issue: "Can't connect to MySQL server"

**Penyebab**: MySQL server tidak running

**Solusi**:
```bash
# Windows: Start MySQL service
net start MySQL80

# macOS: Start via Homebrew
brew services start mysql

# Linux: Start via systemd
sudo systemctl start mysql

# Verifikasi
mysql -u root -p
# Type password, jika bisa login = ok
exit;
```

---

### Issue: "Access denied for user 'root'@'localhost'"

**Penyebab**: Password salah di .env

**Solusi**:
```bash
# Test koneksi manual
mysql -u root -p
# Masukkan password Anda

# Update password di .env jika berbeda
nano .env
# DB_PASSWORD=your_correct_password
```

---

### Issue: "YOLO Model not found"

**Penyebab**: Model belum didownload

**Solusi**:
```bash
# Model akan auto-download saat first run
# Atau manual download:
wget https://github.com/ultralytics/assets/releases/download/v8.0.0/yolov8_final.pt

# Check file ada
ls -la yolov8_final.pt
```

---

### Issue: "dlib installation failed on Windows"

**Penyebab**: Visual C++ Build Tools tidak installed

**Solusi**:
1. Download Visual C++ Build Tools: https://visualstudio.microsoft.com/visual-cpp-build-tools/
2. Install dan restart komputer
3. Jalankan: `pip install dlib`

---

### Issue: "InsightFace model download failed"

**Penyebab**: Internet connection issue atau model URL berubah

**Solusi**:
```bash
# Manual model placement
# 1. Download dari: https://github.com/deepinsight/insightface
# 2. Place di: ~/.insightface/models/
# 3. Restart aplikasi
```

---

## 📦 Upgrade Dependencies

```bash
# Check outdated packages
pip list --outdated

# Upgrade specific package
pip install --upgrade package_name

# Update requirements.txt
pip freeze > requirements.txt

# Untuk production, test sebelum update
```

---

## 🗑️ Cleanup

Jika perlu membersihkan:

```bash
# Remove virtual environment
rm -rf proctoring_env  # macOS/Linux
rmdir /s proctoring_env  # Windows

# Remove cache files
find . -type d -name __pycache__ -exec rm -rf {} +
find . -type f -name "*.pyc" -delete

# Remove temporary files
rm -rf temp/
rm -rf uploads/
```

---

## ✅ Production Checklist

Sebelum deploy ke production:

- [ ] Update SECRET_KEY di .env
- [ ] Set DEBUG=False di app.py
- [ ] Setup database backup
- [ ] Setup SSL certificate
- [ ] Configure Cloudinary untuk production
- [ ] Test semua API endpoints
- [ ] Setup logging
- [ ] Configure firewall rules
- [ ] Setup monitoring
- [ ] Create deployment documentation

---

## 📞 Troubleshooting Support

Jika masih ada masalah:

1. **Check logs**: `tail -f logs/proctoring.log`
2. **Enable debug mode**: `FLASK_ENV=development python app.py`
3. **Check port availability**: `lsof -i :5000` (macOS/Linux)
4. **Inspect network**: `curl http://localhost:5000`

---

## 🎓 Next Steps

1. Baca [README.md](README.md) untuk overview
2. Setup SSL untuk HTTPS
3. Configure Cloudinary integration
4. Buat admin user pertama
5. Run test exam session
6. Configure email notifications
7. Setup backup strategy

---

**Happy Installing! 🎉**

Jika ada pertanyaan, buka issue di GitHub atau hubungi support team.
