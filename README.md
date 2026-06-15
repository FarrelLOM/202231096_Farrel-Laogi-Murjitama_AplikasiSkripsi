# 🎓 Proctoring Application - Sistem Pengawas Ujian Berbasis AI

Aplikasi berbasis web untuk pengawasan ujian online real-time menggunakan teknologi **Face Recognition**, **Gaze Tracking**, dan **YOLO Object Detection**. Dirancang untuk memastikan integritas akademik dengan deteksi pelanggaran secara otomatis.

---

## 📋 Daftar Isi

- [Fitur Utama](#-fitur-utama)
- [Teknologi & Stack](#-teknologi--stack)
- [Persyaratan Sistem](#-persyaratan-sistem)
- [Instalasi](#-instalasi)
- [Konfigurasi](#-konfigurasi)
- [Operasi & Deployment](#-operasi--deployment)
- [Struktur Direktori](#-struktur-direktori)
- [API Reference](#-api-reference)
- [Troubleshooting](#-troubleshooting)
- [Lisensi](#-lisensi)

---

## ✨ Fitur Utama

### 🔐 Autentikasi & Manajemen
- **Admin Panel**: Dashboard lengkap untuk manajemen sesi ujian, peserta, dan hasil
- **Login Peserta**: Sistem login aman dengan validasi OTP
- **Role-Based Access**: Kontrol akses berdasarkan peran admin dan peserta
- **Session Management**: Manajemen sesi ujian dengan kode akhir pembukaan hasil

### 👤 Deteksi Wajah & Biometrik
- **Face Recognition**: Verifikasi identitas peserta menggunakan embedding wajah (InsightFace)
- **Cosine Similarity Matching**: Deteksi ketidakcocokan wajah dengan threshold yang dapat disesuaikan
- **Multi-Face Detection**: Pendeteksian jika ada wajah orang lain selama ujian
- **Gaze Tracking**: Pelacakan arah pandangan untuk deteksi mata menjauh

### 🚨 Deteksi Pelanggaran Real-Time
Sistem mendeteksi dan mencatat pelanggaran berikut secara real-time:
- ❌ **face_mismatch**: Wajah tidak cocok dengan foto referensi
- ❌ **face_not_forward**: Kepala tidak menghadap kamera
- ❌ **multi_face**: Terdeteksi lebih dari satu wajah
- ❌ **eyes_looking_away**: Mata tidak fokus di layar (gaze tracking)
- ❌ **blinking**: Mata berkedip abnormal
- ❌ **no_face_detected**: Wajah tidak terdeteksi di frame

### 📊 Analytics & Reporting
- **Real-Time Violation Logging**: Pencatatan otomatis setiap pelanggaran dengan timestamp
- **Evidence Upload**: Unggah bukti pelanggaran ke Cloudinary dengan secure URL
- **Exam Decision**: Keputusan akhir (JUJUR/CURANG) berdasarkan algoritma penilaian
- **Comprehensive Dashboard**: Visualisasi data pelanggaran dan hasil ujian

### 💾 Database Integration
- **MySQL Database**: Penyimpanan terpusat untuk sesi, peserta, dan pelanggaran
- **Structured Tables**: Skema terstruktur dengan relasi antar tabel
- **Activity Logging**: Pencatatan lengkap setiap aktivitas peserta

### ☁️ Cloud Storage
- **Cloudinary Integration**: Upload otomatis bukti pelanggaran ke cloud
- **Secure URLs**: Link permanen untuk akses bukti kapan saja

---

## 🛠 Teknologi & Stack

### Backend
| Teknologi | Versi | Fungsi |
|-----------|-------|--------|
| Python | 3.10+ | Runtime |
| Flask | 2.3.0 | Web Framework |
| MySQL Connector | 9.5.0 | Database Driver |

### AI/ML
| Library | Versi | Fungsi |
|---------|-------|--------|
| InsightFace | 0.7.3 | Face Recognition & Embedding |
| YOLOv8 | 8.0.196 | Object Detection (Pose Classification) |
| dlib | 19.24.4 | Facial Landmarks & Gaze Tracking |

### Image Processing
| Library | Versi | Fungsi |
|---------|-------|--------|
| OpenCV | 4.10.0.84 | Video Frame Processing |
| NumPy | 1.26.4 | Numerical Computing |
| Pillow | 12.1.0 | Image Manipulation |

### Cloud & Security
| Library | Versi | Fungsi |
|---------|-------|--------|
| Cloudinary | 1.44.1 | Cloud Image Storage |
| python-jose | 3.5.0 | JWT Token Handling |
| cryptography | 46.0.3 | Encryption & Hashing |

### HTTP & Utilities
| Library | Versi | Fungsi |
|---------|-------|--------|
| requests | 2.32.5 | HTTP Client |
| python-dotenv | 1.2.1 | Environment Variables |

---

## 💻 Persyaratan Sistem

### Hardware Minimum
```
CPU: Intel Core i5 / Ryzen 5 atau setara
RAM: 8 GB (16 GB recommended)
GPU: NVIDIA CUDA (optional, untuk performa lebih baik)
Storage: 5 GB
```

### Software
```
Python: 3.10 atau lebih baru
MySQL Server: 5.7 atau lebih baru
Node.js: Untuk development (optional)
```

### Network
- Koneksi internet stabil (minimum 2 Mbps)
- Port yang tersedia: 5000 (Flask), 3306 (MySQL)

---

## 📦 Instalasi

### Step 1: Persiapan Lingkungan

#### Opsi A: Menggunakan conda (Recommended)
```bash
# Buat conda environment
conda create -n proctoring python=3.10 -y
conda activate proctoring
```

#### Opsi B: Menggunakan venv
```bash
# Buat virtual environment
python -m venv proctoring_env

# Aktivasi (Windows)
proctoring_env\Scripts\activate

# Aktivasi (macOS/Linux)
source proctoring_env/bin/activate
```

### Step 2: Clone & Setup Project
```bash
# Clone repository
git clone https://github.com/yourusername/proctoring-app.git
cd proctoring-app

# Install dependencies
pip install -r requirements.txt
```

### Step 3: Download Model Files
Aplikasi memerlukan model AI pre-trained:

```bash
# Model akan diunduh otomatis saat first run
# InsightFace (buffalo_l): ~350 MB
# YOLO v8: ~60 MB
# dlib shape predictor: ~50 MB

# Total ukuran model: ~460 MB
```

### Step 4: Setup Database

#### 4a. Buat Database MySQL
```sql
CREATE DATABASE proctoring_app;
USE proctoring_app;
```

#### 4b. Run Migration Script
```bash
# Buka file database_create_table.sql atau DATABASE_MIGRATION.sql
# Execute di MySQL client atau terminal

mysql -u root -p proctoring_app < DATABASE_MIGRATION.sql
```

#### 4c. Verifikasi Tabel (Optional)
```bash
python verify_structure.py
python verify_violations_db.py
```

---

## ⚙️ Konfigurasi

### Step 1: Setup Environment Variables

Buat file `.env` di root directory:

```bash
# Server Configuration
SERVER_HOST=localhost
PORT=5000
SECRET_KEY=your-secret-key-here-change-this

# Database Configuration
DB_HOST=localhost
DB_PORT=3306
DB_NAME=proctoring_app
DB_USER=root
DB_PASSWORD=your_password_here

# Admin Credentials
ADMIN_USERNAME=admin
ADMIN_PASSWORD=your_admin_password

# Cloudinary Configuration
CLOUDINARY_CLOUD_NAME=your_cloudinary_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
CLOUDINAR_MAIN_FOLDER=proctoring_app/

# AI/ML Configuration
YOLO_MODEL=yolov8_final.pt
CONF_THRES=0.5
FRAME_WIDTH=640
FRAME_HEIGHT=480
YOLO_INTERVAL=2
FACE_INTERVAL=10
GAZE_INTERVAL=3
SIM_THRESHOLD=0.45
VIOLATION_DURATION=6

# Optional: GPU Configuration
USE_GPU=true
GPU_ID=0
```

### Step 2: Konfigurasi Database

Sesuaikan kredensial database di `.env` sesuai setup lokal Anda.

### Step 3: Setup Cloudinary (Optional)

Jika ingin menggunakan cloud storage:

1. Daftar di [Cloudinary](https://cloudinary.com)
2. Copy API credentials
3. Paste ke `.env`

Jika tidak ingin cloud storage, sistem tetap berfungsi dengan penyimpanan lokal.

---

## 🚀 Operasi & Deployment

### Development Mode

#### Local Testing
```bash
# Aktifkan virtual environment
conda activate proctoring
# atau
source proctoring_env/bin/activate

# Run development server
python app.py

# Akses di browser
# Admin: http://localhost:5000/admin/login
# Peserta: http://localhost:5000/login
```

#### Test Suite
```bash
# Test koneksi database
python check_current_state.py

# Test exam URLs
python check_exam_urls.py

# Test system (comprehensive)
python test_system.py

# Test violation API
python test_violation_api_fixed.py

# Test database violations
python test_violations_table.py
```

### Production Deployment

#### Opsi 1: Vercel (Recommended untuk Cloud)

**⚠️ CATATAN PENTING**: Vercel tidak support aplikasi Flask dengan heavy ML models. Gunakan untuk frontend/API ringan saja.

```bash
# Install Vercel CLI
npm install -g vercel

# Login
vercel login

# Deploy (dari directory project)
vercel

# Konfigurasi environment variables di Vercel dashboard
```

#### Opsi 2: Heroku

```bash
# Install Heroku CLI
npm install -g heroku

# Login
heroku login

# Buat app
heroku create your-app-name

# Set environment variables
heroku config:set DB_HOST=your-db.com
heroku config:set CLOUDINARY_CLOUD_NAME=your-name

# Deploy
git push heroku main
```

#### Opsi 3: AWS / Digital Ocean / VPS (Recommended)

**A. Setup Server**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install python3.10 python3-pip mysql-server -y

# Clone repository
git clone https://github.com/yourusername/proctoring-app.git
cd proctoring-app
```

**B. Setup Python Environment**
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**C. Setup MySQL**
```bash
sudo mysql -u root
CREATE DATABASE proctoring_app;
# Run migration SQL
```

**D. Run dengan Gunicorn (Production)**
```bash
# Install Gunicorn
pip install gunicorn

# Run server
gunicorn -w 4 -b 0.0.0.0:5000 app:app
```

**E. Setup Nginx (Reverse Proxy)**
```bash
sudo apt install nginx -y

# Create Nginx config
sudo nano /etc/nginx/sites-available/proctoring

# Content:
server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

# Enable site
sudo ln -s /etc/nginx/sites-available/proctoring /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

**F. Setup SSL (HTTPS)**
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d your-domain.com
```

**G. Automate dengan Systemd**
```bash
# Create service file
sudo nano /etc/systemd/system/proctoring.service

# Content:
[Unit]
Description=Proctoring Application
After=network.target

[Service]
User=www-data
WorkingDirectory=/home/username/proctoring-app
ExecStart=/home/username/proctoring-app/venv/bin/gunicorn -w 4 -b 127.0.0.1:5000 app:app
Restart=always

[Install]
WantedBy=multi-user.target

# Enable service
sudo systemctl enable proctoring
sudo systemctl start proctoring
sudo systemctl status proctoring
```

---

## 📁 Struktur Direktori

```
proctoring-app/
│
├── app.py                                    # Main Flask application
├── face_analyzer.py                          # Face analysis engine wrapper
├── realtime_proctoring_stable_highfps.py    # Core AI/ML proctoring logic
│
├── gaze_tracking/                            # Gaze tracking module
│   ├── __init__.py
│   ├── calibration.py
│   ├── eye.py
│   ├── pupil.py
│   ├── gaze_tracking.py
│   └── trained_models/
│       └── shape_predictor_68_face_landmarks.z*
│
├── static/                                   # Frontend assets
│   ├── style.css                            # Styling
│   ├── exam-monitor.js                      # Main exam JavaScript
│   ├── webcam-analyzer.js                   # Webcam processing
│   └── [other JS files]
│
├── templates/                                # HTML templates
│   ├── admin_login.html
│   ├── admin_dashboard.html
│   ├── admin_session.html
│   ├── admin_participant.html
│   ├── admin_results.html
│   ├── login.html
│   ├── participant_dashboard.html
│   ├── proctored-exam.html
│   └── [other templates]
│
├── database/                                 # Database scripts
│   ├── database_create_table.sql
│   ├── DATABASE_MIGRATION.sql
│   └── UPDATE_VIOLATION_EVIDENCE_SCHEMA.sql
│
├── tests/                                    # Test scripts
│   ├── test_system.py
│   ├── test_violation_api.py
│   ├── test_violations_table.py
│   └── [other test files]
│
├── yolov8_final.pt                          # YOLO model (auto-downloaded)
├── requirements.txt                          # Python dependencies
├── .env                                      # Environment configuration
├── README.md                                 # This file
└── .gitignore                               # Git ignore rules
```

---

## 🔌 API Reference

### Authentication Endpoints

#### Admin Login
```http
POST /admin/login
Content-Type: application/x-www-form-urlencoded

username=admin&password=your_password

Response: 302 Redirect to /admin/dashboard
```

#### Participant Login
```http
POST /login
Content-Type: application/x-www-form-urlencoded

exam_id=1&participant_number=P001

Response: 302 Redirect to /participant/dashboard
```

### Session Management

#### Get All Sessions
```http
GET /admin/sessions
Authorization: Required (Admin)

Response:
[
  {
    "id": 1,
    "session_name": "Ujian Algoritma",
    "exam_url": "http://exam.com/test",
    "created_at": "2024-01-20T10:00:00"
  }
]
```

### Violation Recording

#### Record Single Violation
```http
POST /api/record-violation
Content-Type: application/json

{
  "participant_id": 1,
  "session_id": 1,
  "violation_type": "face_mismatch",
  "timestamp": "2024-01-20T10:05:00.000Z",
  "evidence_image": "data:image/jpeg;base64,..."
}

Response:
{
  "status": "success",
  "violation_id": 123,
  "message": "Violation recorded"
}
```

#### Get Exam Result
```http
GET /api/exam-result/{participant_id}

Response:
{
  "participant_id": 1,
  "total_violations": 3,
  "violation_types": ["face_mismatch", "eyes_looking_away"],
  "exam_decision": "CURANG",
  "result_percentage": 25
}
```

### Webcam Processing

#### Initialize Webcam
```http
POST /api/webcam-init
Content-Type: application/json

{
  "participant_id": 1,
  "face_photo_url": "http://cloudinary.com/image.jpg"
}
```

#### Process Frame
```http
POST /api/process-frame
Content-Type: application/json

{
  "frame": "data:image/jpeg;base64,...",
  "participant_id": 1
}

Response:
{
  "violations": ["face_mismatch"],
  "face_similarity": 0.87,
  "frame_count": 150
}
```

---

## 🐛 Troubleshooting

### Database Issues

#### Error: "Couldn't parse requirements.txt"
**Solusi**: Gunakan format plain requirements.txt (bukan output conda):
```bash
# ❌ SALAH (format conda export)
# Name    Version    Build    Channel
numpy    1.26.4    pypi_0    pypi

# ✅ BENAR (format pip)
numpy==1.26.4
```

#### Error: "No module named 'mysql.connector'"
```bash
pip install mysql-connector-python==9.5.0
```

### Model Loading Issues

#### YOLO Model Not Found
```bash
# Download manual
wget https://github.com/ultralytics/assets/releases/download/v8.0.0/yolov8_final.pt -O yolov8_final.pt
```

#### dlib Installation Failed
```bash
# Windows
pip install dlib-19.24.4-cp310-cp310-win_amd64.whl

# macOS
brew install dlib

# Linux
sudo apt install libopenblas-dev
pip install dlib
```

### Cloudinary Issues

#### Error: "Invalid Cloudinary credentials"
- Verifikasi credentials di `.env`
- Pastikan API key dan secret benar
- Check folder path di CLOUDINAR_MAIN_FOLDER

### Performance Issues

#### High CPU Usage
- Sesuaikan `YOLO_INTERVAL` (naikan ke 5-10)
- Sesuaikan `FACE_INTERVAL` (naikan ke 20-30)
- Reduce frame resolution: `FRAME_WIDTH=480, FRAME_HEIGHT=360`

#### Memory Leak
```bash
# Monitor
python -m memory_profiler app.py

# Optimize
# - Batasi model cache
# - Clear old sessions regularly
```

### Deployment Issues

#### Vercel Deployment Error
**Masalah**: ML models terlalu besar untuk Vercel (~460 MB)

**Solusi**: Gunakan serverless functions dengan external model storage
```bash
# Atau deploy di VPS/AWS
```

#### 502 Bad Gateway (Gunicorn)
```bash
# Increase workers
gunicorn -w 8 -b 0.0.0.0:5000 app:app

# Increase timeout
gunicorn -w 4 -b 0.0.0.0:5000 --timeout 120 app:app
```

---

## 📊 Database Schema Overview

### Key Tables

#### `sessions`
```sql
- id (PK)
- session_name
- exam_url
- exam_end_code
- start_time
- end_time
- created_at
```

#### `participants`
```sql
- id (PK)
- session_id (FK)
- participant_number
- name
- email
- face_photo_url
- login_status
- created_at
```

#### `violations`
```sql
- id (PK)
- participant_id (FK)
- violation_type
- duration_seconds
- start_time
- end_time
- created_at
```

#### `violation_evidence`
```sql
- id (PK)
- violation_id (FK)
- evidence_url (Cloudinary URL)
- created_at
```

---

## 🔒 Security Considerations

1. **Environment Variables**: Jangan commit `.env` ke git
2. **Database Credentials**: Gunakan credentials unik per environment
3. **JWT Tokens**: Set SECRET_KEY yang kuat di production
4. **HTTPS**: Selalu gunakan SSL di production
5. **API Rate Limiting**: Implement rate limiting untuk production
6. **Input Validation**: Semua input divalidasi di backend

---

## 📝 Git Workflow

```bash
# Clone repo
git clone https://github.com/yourusername/proctoring-app.git

# Create feature branch
git checkout -b feature/new-feature

# Make changes & commit
git add .
git commit -m "Add new feature"

# Push to remote
git push origin feature/new-feature

# Create Pull Request
# -> Review & Merge
```

### .gitignore Template
```
# Environment
.env
.env.local

# Virtual Environment
proctoring_env/
venv/

# Cache
__pycache__/
*.pyc
*.pyo

# IDE
.vscode/
.idea/
*.swp

# Database
*.db
*.sqlite

# Models (too large)
*.pt
*.pth
*.onnx

# Uploads
uploads/
temp/

# OS
.DS_Store
Thumbs.db
```

---

## 📚 Referensi & Resources

- **Flask**: https://flask.palletsprojects.com
- **InsightFace**: https://github.com/deepinsight/insightface
- **YOLOv8**: https://github.com/ultralytics/ultralytics
- **MySQL**: https://dev.mysql.com/doc
- **Cloudinary**: https://cloudinary.com/documentation

---

## 👥 Kontribusi

Kontribusi sangat diterima! Silakan:

1. Fork repository
2. Buat feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push ke branch (`git push origin feature/AmazingFeature`)
5. Buat Pull Request

---

## 📞 Support & Contact

Jika mengalami masalah atau butuh bantuan:

- **Issues**: Buka issue di GitHub
- **Email**: your-email@example.com
- **Documentation**: Lihat folder `docs/`

---

## 📄 Lisensi

Project ini dilisensikan di bawah MIT License - lihat file `LICENSE` untuk detail.

---

## 🎯 Roadmap

- [ ] Mobile app (React Native)
- [ ] Real-time analytics dashboard
- [ ] Advanced ML models
- [ ] Multi-language support
- [ ] Accessibility improvements
- [ ] API documentation (Swagger)

---

**Last Updated**: 2026-06-15  
**Version**: 1.0.0  
**Status**: ✅ Production Ready

---

<div align="center">

Made with ❤️ for Academic Integrity

[⬆ Back to top](#-proctoring-application---sistem-pengawas-ujian-berbasis-ai)

</div>
