# 🌍 GeoLens — AI-Powered Satellite Image Analysis and Classification System

**GeoLens** is a comprehensive platform for analyzing satellite imagery using **Deep Learning** and **Computer Vision** technologies. The system supports **Semantic Segmentation**, **Object Classification**, **Crop Health Analysis**, and **Project Management** through an interactive web-based interface.

---

## 📋 Table of Contents

- [Key Features](#-key-features)
- [System Architecture](#-system-architecture)
- [System Requirements](#-system-requirements)
- [Installation and Setup](#-installation-and-setup)
- [Project Walkthrough](#-project-walkthrough)
- [Workflow](#-workflow)
- [Users and Permissions](#-users-and-permissions)
- [Database](#-database)
- [API](#-api)
- [Python AI Scripts](#-python-ai-scripts)
- [Known Issues](#-known-issues)
- [Future Improvements](#-future-improvements)

---

## ✨ Key Features

| Feature | Description |
|---------|-------------|
| 🔐 **Authentication System** | User registration and login powered by Laravel Breeze |
| 📁 **Project Management** | Create, edit, and delete projects. Each project has its own satellite images and annotation classes. |
| 📸 **Satellite Image Upload** | Upload multi-band `.tif` satellite images (minimum of 4 bands: R, G, B, NIR). |
| 🏷️ **Annotation Class Management** | Create and manage annotation classes (e.g., Building, Vegetation, Water, Road) for each project. |
| 🎯 **Semantic Segmentation (SAM)** | Segment satellite images using Meta AI's **Segment Anything Model (SAM)**. |
| 🤖 **Object Classification (ResNet-50)** | Classify segmented regions using a **ResNet-50** model trained on the **EuroSAT** dataset (10 classes, 97.7% accuracy). |
| 🌱 **Crop Health Analysis** | Calculate NDVI, generate advanced vegetation statistics (Healthy, Stressed, Unhealthy), and visualize results with charts. |
| 📊 **Dashboard** | Monitor overall project statistics, uploaded images, classifications, and crop health reports. |
| 📈 **Health Reports** | Interactive Pie Charts and Bar Charts powered by Chart.js with detailed crop health insights. |
| 🗺️ **GeoTIFF Metadata Extraction** | Automatically extract image metadata such as Width, Height, Bands, and CRS using Rasterio. |

---

## 🏗️ System Architecture

```text
annotation-project/
│
├── backend/                          # Laravel 12 (PHP 8.2)
│   ├── app/
│   │   ├── Http/Controllers/         # 7 Controllers (Web + API Bridge)
│   │   ├── Models/                   # 5 Models (Eloquent ORM)
│   │   └── ...
│   ├── resources/views/              # 12 Blade Views (Tailwind CSS + Chart.js)
│   ├── routes/web.php                # 17 Routes (Authentication + CRUD + Python Bridge)
│   ├── database/migrations/          # 9 Migrations (5 Project-Specific Migrations)
│   └── ...
│
├── geo_processor.py                  # NDVI Calculation, Heatmap Generation, and Band Statistics
├── sam__predectorr.py                # SAM Segmenter (Click-Based Segmentation)
├── classifier.py                     # ResNet-50 Classifier (EuroSAT, 10 Classes)
├── projects_manager.py               # File-Based Project Management
├── sam_update.py                     # Optional SAM Update with COG Pipeline
├── sam_test.py                       # SAM Testing Script
│
└── checkpoint/                       # Model Weight Files
    ├── classifier_weights.pth        # ResNet-50 Weights (~94 MB) ✅ Available
    └── sam_vit_b_01ec64.pth          # SAM Weights (~375 MB) ❌ Missing
```

### Technology Stack

| Layer | Technology |
|--------|------------|
| **Backend Framework** | Laravel 12 (PHP 8.2.20) |
| **Frontend** | Blade + Tailwind CSS + Chart.js + jQuery |
| **Build Tool** | Vite 7 |
| **Database** | MySQL (annotation_db) |
| **AI/ML** | Python 3.8.8 + PyTorch (CPU) + Rasterio + Segment Anything |
| **Authentication** | Laravel Breeze (Blade Stack) |

---

## 📦 System Requirements

### Core Requirements

- **PHP** ≥ 8.1 (Current Version: 8.2.20)
- **Composer** (PHP Package Manager)
- **Node.js** ≥ 18 (Installed)
- **MySQL** (Running on `127.0.0.1:3306`)
- **Python** 3.8+ (Current Version: 3.8.8)

### Required Python Packages

```bash
pip install torch torchvision --index-url https://download.pytorch.org/whl/cpu
pip install rasterio numpy pillow matplotlib opencv-python
```
### Required for SAM Segmentation

- Download the `sam_vit_b_01ec64.pth` checkpoint file (~375 MB) from:
  https://dl.fbaipublicfiles.com/segment_anything/sam_vit_b_01ec64.pth
  ```
### Optional

- **GDAL** (Required only for running the COG pipeline in `sam_update.py`)

---

# 🚀 Installation & Setup

## 1. Clone the Repository

```
bash
git clone https://github.com/<your-username>/<repo-name>.git
cd annotation-project
```
http://127.0.0.1:8000/
```
2. Set Up the Laravel Backend
bash
cd backend

# Install PHP dependencies
composer install

# Install JavaScript dependencies
npm install

# Build frontend assets (CSS/JS)
npm run build

# Create the environment file
copy .env.example .env
Update the .env file:
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=annotation_db
DB_USERNAME=root
DB_PASSWORD=
Then run:
# Generate the application key
php artisan key:generate

# Create the database (using MySQL or phpMyAdmin)
mysql -u root -e "CREATE DATABASE IF NOT EXISTS annotation_db"

# Run database migrations
php artisan migrate

# Create a symbolic link for uploaded files
php artisan storage:link

# Start the Laravel development server
php artisan serve
3. Start Vite Development Server (Optional)
cd backend
npm run dev
4. Open the Application
http://127.0.0.1:8000/
5. Download the SAM Checkpoint
To enable image segmentation, download:
https://dl.fbaipublicfiles.com/segment_anything/sam_vit_b_01ec64.pth
Place it in:
annotation-project/checkpoint/sam_vit_b_01ec64.pth
```
🧭 Project Walkthrough
Welcome Page → GET /
<img src="prototype demo/Screenshot 2026-06-20 210126.png" width="600" alt="Welcome Page">

The GeoLens welcome page introduces the project, provides a brief overview, and includes a Get Started button for user registration.

Authentication → GET /login | GET /register
<img src="prototype demo/Screenshot 2026-06-20 210229.png" width="600" alt="Register">

A secure authentication system built with Laravel Breeze (Blade Stack), allowing users to register and log in.

Create a Project → GET /projects/create | POST /projects
<img src="prototype demo/Screenshot 2026-06-20 210743.png" width="600" alt="Projects">

Create a new project by providing a project name and an optional description.

Each project includes:

Satellite Images (.tif files)
Annotation Classes (e.g., Building, Vegetation, Water, Road)
Annotations for image segmentation and classification
### Project Overview & Image Upload → `GET /projects/{id}`
<img src="prototype demo/Screenshot 2026-06-20 210928.png" width="600" alt="Project Details">

- Project information
- Uploaded images table (including Width, Height, Bands, and CRS)
- Upload new `.tif` satellite images
- Manage annotation classes (Add/Delete)
- **Annotate** button for each uploaded image
- **Health Report** button for crop health analysis

### Annotation Workspace → `GET /projects/{project}/annotate/{image}`

- Display the satellite image on an interactive canvas
- Select an annotation class
- Click on the image to perform **SAM Segmentation** via AJAX
- Display the generated segmentation mask
- **Undo** the last segmentation
- **Zoom In / Zoom Out**
- Classify the selected region using **ResNet-50**
- **Export GeoJSON** to download annotations
- **Analyze Health** to evaluate crop health within the selected region

### Crop Health Report → `GET /projects/{project}/health-report`

- Pie Chart showing **Healthy %, Stressed %, and Unhealthy %**
- Bar Chart comparing crop health areas
- Historical reports table (Total Area, Healthy, Stressed, Unhealthy)
- Delete previous reports

### Dashboard → `GET /dashboard`

- Summary cards displaying projects, images, classifications, and health reports
- Table showing the latest five projects
- **Pie Chart** illustrating project status
- **Bar Chart** showing the number of images per project

---

# 🔄 Workflow

## 📊 Complete System Workflow

```text
Guest User
    │
    ├─→ Welcome Page (/) ←── Landing page
    │
    ├─→ Register ←── Create a new account
    │
    └─→ Login ←── Sign in
         │
         ▼
Authenticated User
         │
         ├─→ Dashboard ←── Overview (Statistics, Recent Projects, Health Reports)
         │
         ├─→ Projects List ←── View all projects
         │      │
         │      ├─→ Create Project ←── Create a new project (Name + Description)
         │      │
         │      ├─→ Project Details
         │      │      │
         │      │      ├─→ Upload Image (.tif) ←── Upload a multi-band satellite image
         │      │      │
         │      │      ├─→ Add Annotation Classes ←── Define class names and colors
         │      │      │
         │      │      ├─→ Annotate Workspace
         │      │      │      │
         │      │      │      ├─→ Select Annotation Class
         │      │      │      ├─→ Click on Canvas
         │      │      │      ├─→ SAM Segmentation
         │      │      │      ├─→ ResNet-50 Classification
         │      │      │      ├─→ Save Annotation
         │      │      │      ├─→ Undo / Zoom
         │      │      │      └─→ Export GeoJSON
         │      │      │
         │      │      └─→ Crop Health Report
         │      │             │
         │      │             ├─→ Chart.js Pie Chart (Healthy / Stressed / Unhealthy)
         │      │             ├─→ Chart.js Bar Chart (Area Comparison)
         │      │             └─→ Previous Reports Table
         │      │
         │      └─→ Edit / Delete Project
         │
         ├─→ AI Assistant ←── Ask questions in natural language
         │
         └─→ Profile ←── Update profile information and change password
```

## 🔄 Data Flow

```text
1. User uploads a multi-band (.tif) satellite image
        │
2. Laravel invokes Python (Rasterio) to extract image metadata:
   Width, Height, Bands, and CRS
        │
3. User selects an annotation class and clicks on the image
        │
4. Laravel executes Python (`sam__predectorr.py`) via shell_exec()
        │
5. SAM returns:
   mask_image, polygon_coordinates, bounding_box, and area
        │
6. User selects "Classify"
   → ResNet-50 (EuroSAT)
   → Predicted label + confidence score
        │
7. User selects "Analyze Health"
   → geo_processor.py
   → NDVI statistics and crop health analysis
        │
8. All results are stored in MySQL
   (annotations & crop_health_results tables)
```

---

# 👤 Users & Permissions

## Default Users (Seed Data)

| User | Email | Password | Role |
|------|-------|----------|------|
| 👑 **Ahmed (Administrator)** | `admin@geolens.com` | `password` | Administrator — Manages projects and users |
| 🔬 **Sara (Analyst)** | `analyst@geolens.com` | `password` | Analyst — Creates projects and analyzes satellite images |
| 👁️ **Khaled (Viewer)** | `viewer@geolens.com` | `password` | Viewer — Can only view projects and reports |

## Current Permission System

The current implementation uses **Laravel Breeze Authentication** without a dedicated Role-Based Access Control (RBAC) system. Therefore, all authenticated users currently have the same permissions.

| Action | Administrator | Analyst | Viewer |
|---------|:-------------:|:-------:|:------:|
| Create Projects | ✅ | ✅ | ✅ |
| View Projects | ✅ | ✅ | ✅ |
| Edit/Delete Own Projects | ✅ | ✅ | ✅ |
| Upload Images | ✅ | ✅ | ✅ |
| Manage Annotation Classes | ✅ | ✅ | ✅ |
| Create Annotations | ✅ | ✅ | ✅ |
| View Health Reports | ✅ | ✅ | ✅ |
| Access AI Features | ✅ | ✅ | ✅ |

> **Note:** Each user can only access their own projects using `Auth::id()`. A dedicated administrator authorization system has not yet been implemented.

### Create a New User

```bash
php artisan tinker
> \App\Models\User::create([
    'name' => 'User Name',
    'email' => 'user@example.com',
    'password' => bcrypt('password'),
    'email_verified_at' => now(),
]);
```

---

# 🗄️ Database

```sql
-- annotation_db (MySQL)

users                    -- id, name, email, password
projects                 -- id, user_id, name, description
image_uploads            -- id, project_id, user_id, original_name, file_path,
                         --    file_size, width, height, bands, crs
annotation_classes       -- id, project_id, name, color
annotations              -- id, image_upload_id, annotation_class_id, user_id,
                         --    mask_data (BLOB), polygon_coordinates (JSON),
                         --    bbox, area_pixels, area_m2,
                         --    classification_label, classification_confidence,
                         --    geo_metadata (JSON)
crop_health_results      -- id, project_id, image_upload_id,
                         --    total/healthy/stressed/unhealthy area (m²),
                         --    healthy/stressed/unhealthy_percentage,
                         --    overall_status, raw_stats (JSON)
```
 خطوة بخطوة
**Relationships:**
- `User` → hasMany `projects`, `image_uploads`, and `annotations`
- `Project` → hasMany `image_uploads`, `annotation_classes`, and `crop_health_results`
- `ImageUpload` → hasMany `annotations`
- `AnnotationClass` → hasMany `annotations`

---

# 🌐 API Documentation

## Web Routes (19 Routes)

| Method | Route | Controller | Purpose |
|--------|-------|-----------|---------|
| GET | `/` | WelcomeController | Welcome page |
| GET/POST | `/login` | Auth | User login |
| GET/POST | `/register` | Auth | User registration |
| GET | `/dashboard` | DashboardController | Dashboard |
| GET | `/projects` | ProjectController@index | List all projects |
| GET | `/projects/create` | ProjectController@create | Create a new project |
| POST | `/projects` | ProjectController@store | Store a new project |
| GET | `/projects/{id}` | ProjectController@show | View project details |
| GET | `/projects/{id}/edit` | ProjectController@edit | Edit project |
| DELETE | `/projects/{id}` | ProjectController@destroy | Delete project |
| POST | `/projects/{id}/images/upload` | ImageUploadController@upload | Upload a satellite image |
| GET | `/projects/{id}/annotate/{image}` | ImageUploadController@annotate | Annotation workspace |
| POST | `/projects/{id}/classes` | AnnotationClassController@store | Create an annotation class |
| DELETE | `/projects/{id}/classes/{class}` | AnnotationClassController@destroy | Delete an annotation class |
| POST | `/projects/{id}/segment` | PythonBridgeController@segment | Run SAM segmentation |
| POST | `/projects/{id}/classify` | PythonBridgeController@classify | Run ResNet-50 classification |
| POST | `/projects/{id}/analyze-health` | PythonBridgeController@analyzeHealth | Perform NDVI analysis |
| GET | `/projects/{id}/health-report` | DashboardController@healthReport | Crop health report |
| GET | `/assistant` | AssistantController@index | AI Assistant page |
| POST | `/assistant/ask` | AssistantController@ask | AI Assistant API |

## AJAX Endpoints (Annotation Workspace)

```javascript
// Run SAM Segmentation
POST /projects/{id}/segment
Body: { image_path, x, y }
→ Response: { mask_path, polygon_coordinates, ... }

// Classify the selected region
POST /projects/{id}/classify
Body: { image_path, x, y }
→ Response: { label, confidence }

// Analyze crop health
POST /projects/{id}/analyze-health
Body: { image_path }
→ Response: { healthy_percentage, stressed_percentage, ... }

// Save annotation
POST /projects/{id}/annotations
Body: { image_upload_id, annotation_class_id, polygon_coordinates, ... }
→ JSON Response

// Delete annotation
POST /projects/{id}/annotations
Header: X-HTTP-Method-Override: DELETE
```

---

# 🐍 Python AI Scripts

## `geo_processor.py` — GeoTIFF Processing

```python
from geo_processor import process_geotiff

stats = process_geotiff("image.tif")
# {
#   "ndvi_mean": 0.34,
#   "ndvi_std": 0.12,
#   "red_mean": 120.5,
#   "nir_mean": 200.3,
#   "rgb_path": "output_rgb.jpg",
#   "heatmap_path": "output_ndvi_heatmap.jpg"
# }
```

**Features**

- Supports **4-band** satellite imagery (Red, Green, Blue, NIR)
- Calculates **NDVI** using:

```
NDVI = (NIR - Red) / (NIR + Red)
```

- Generates an NDVI heatmap using the OpenCV **JET** colormap.

---

## `sam__predectorr.py` — SAM Segmenter

```python
from sam__predectorr import AdvancedSAMSegmenter

segmenter = AdvancedSAMSegmenter()
result = segmenter.segment("image.tif", x=500, y=300)
# { mask, polygon_coordinates, bbox, area_pixels, area_m2 }
```

**Features**

- Powered by Meta AI's **Segment Anything Model (SAM)**
- Click-based image segmentation
- Extracts polygons and bounding boxes
- Supports GeoJSON export with geographic coordinates
- **Requires** the checkpoint file:

```
checkpoint/sam_vit_b_01ec64.pth
```

---

## `classifier.py` — ResNet-50 Classifier

```python
from classifier import EuroSATResNetClassifier

clf = EuroSATResNetClassifier()
label, confidence = clf.classify_patch("patch.jpg")
# label: "Residential", confidence: 0.977
```

**Features**

- Built using **ResNet-50** (`torchvision.models.resnet50`)
- Pre-trained on the **EuroSAT** dataset (10 land-cover classes)

```
AnnualCrop
Forest
HerbaceousVegetation
Highway
Industrial
Pasture
PermanentCrop
Residential
River
SeaLake
```

- Classification accuracy: **97.7%**
- Model weights available in:

```
checkpoint/classifier_weights.pth
```

(~94 MB)

---

## `projects_manager.py` — Project Management

- File-based CRUD operations for projects
- Stores project information as JSON files
- Each project contains:
  - Name
  - Description
  - Images
  - Annotation Classes

---

## `sam_update.py` — Optional SAM Update

- Cloud Optimized GeoTIFF (COG) pipeline
- Converts GeoTIFF images into COG format using GDAL
- Merges adjacent segmented regions

---

# ⚠️ Known Issues

## 🔴 Blocker — Missing SAM Checkpoint

```
sam_vit_b_01ec64.pth (~375 MB) is required to run sam__predectorr.py
```

**Solution**

Download it from:

```
https://dl.fbaipublicfiles.com/segment_anything/sam_vit_b_01ec64.pth
```

Then place it in:

```
checkpoint/sam_vit_b_01ec64.pth
```

---

## 🔴 Blocker — GDAL Not Installed on Windows

`sam_update.py` attempts to create `/tmp/cog_outputs` (a Linux directory) and requires GDAL.

---

## 🟡 Warning — Deprecated APIs in `classifier.py`

```python
# Uses pretrained=False (deprecated in torchvision 0.19)
# Uses torch.load without weights_only=True
```

A minor update is recommended for compatibility with newer PyTorch versions.

---

## 🟡 Warning — Broken Python Virtual Environment

The `annotation_env` virtual environment path is unavailable, so the system falls back to the default Python installation.

---

## 🟡 Warning — Legacy Streamlit Application

The legacy `app.py` (Streamlit) still runs after installing dependencies but is outdated compared to the Laravel-based interface.

---

# 🧪 Testing

## Laravel Backend

```bash
cd backend
php artisan serve
```

Open:

```
http://127.0.0.1:8000/
```

The Welcome Page should appear successfully.

---

## Route Testing

All routes should return **HTTP 200**.

```
GET  /                 → Welcome Page
GET  /login            → Login Page
GET  /register         → Registration Page
GET  /dashboard        → Dashboard (redirects to Login if unauthenticated)
GET  /projects         → Projects List
GET  /projects/create  → Create Project Page
GET  /forgot-password  → Forgot Password Page
```

---

## Python AI Testing

```bash
# Test geo_processor.py
python geo_processor.py

# Test classifier.py
python -c "from classifier import EuroSATResNetClassifier; c=EuroSATResNetClassifier(); print('✅ Classifier loaded')"

# Requires checkpoint/classifier_weights.pth

# Test SAM
python -c "from sam__predectorr import AdvancedSAMSegmenter; s=AdvancedSAMSegmenter(); print('✅ SAM loaded')"

# Returns FileNotFoundError if the SAM checkpoint is missing.
```

---

## Database Testing

```bash
cd backend
php artisan tinker

> \App\Models\User::count()
> \App\Models\Project::count()
> \App\Models\ImageUpload::count()
```

---

# 📈 Future Improvements

1. Download and integrate the **SAM checkpoint** (`sam_vit_b_01ec64.pth`) to enable image segmentation.
2. Perform comprehensive end-to-end testing using real multi-band GeoTIFF images.
3. Containerize the application with **Docker Compose** for one-command deployment.
4. Develop a fully **RESTful JSON API** for integration with external applications.
5. Enhance the user interface with improved responsiveness and Dark Mode support.
6. Update `classifier.py` to replace deprecated APIs with modern PyTorch implementations.
7. Add **CUDA/GPU acceleration** for faster AI inference.
8. Enable the **Cloud Optimized GeoTIFF (COG)** pipeline by installing and configuring GDAL.

---

# 👨‍💻 Project Information

- **Version:** 1.0.0
- **PHP:** 8.2.20
- **Laravel:** 12
- **Python:** 3.8.8
- **Database:** MySQL 8.0+
- **AI Models:** Segment Anything Model (SAM) + ResNet-50 (EuroSAT)

---

# 👥 Development Team

- **Alshimaa Alaa Mohamed Elsyed**
- **Rana Ibrahim Ahmed Khattab**
- **Sara Mohamed Ahmed Mohamed**
- **Mariam Mamdouh Mostafa Amer**
- **Manar Mahmoud Arafa Mostafa**
---
  
# 📄 License

This project is intended for **educational and research purposes**.

The AI models included in this project are distributed under their respective licenses:

- **SAM (Meta AI)** — MIT License
- **EuroSAT Dataset** — MIT License
