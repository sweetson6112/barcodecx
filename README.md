# 📦 CDRSL Barcode & Expiry Management System

> **Created by Sweetson Joseph**  
> Inward Inspection · Barcode Verification · Shelf Life Tracking

---

## 📋 Overview

A full-featured Streamlit web application for warehouse inward inspection that:

- **Auto-generates Document Numbers** in the series `CDRSL/BARCODE/001`
- **Manages Users** (verifiers, supervisors, admins)
- **Uploads Barcode Master** from Excel/CSV
- **Captures Header Details** for each inward session
- **Scans Barcodes** and compares against master, handles new items
- **Tracks Expiry & Shelf Life** using the formula: `(Expiry − Inward Date) / (Expiry − Mfg Date) × 100%`
- **Generates Excel Reports** with full header and line detail
- **Supports session cancellation** at any point

---

## 🗂️ Project Structure

```
barcode_app/
├── app.py                          # Main Streamlit application
├── utils/
│   ├── __init__.py
│   ├── db.py                       # Supabase database helpers
│   ├── shelf_life.py               # Shelf life calculation logic
│   └── report.py                   # Excel report generation
├── .streamlit/
│   ├── config.toml                 # App theme & server config
│   └── secrets.toml.template       # Credentials template
├── supabase_schema.sql             # Full DB schema (run in Supabase SQL Editor)
├── requirements.txt
├── .gitignore
└── README.md
```

---

## 🚀 Setup Instructions

### Step 1 — Supabase Setup

1. Go to [https://supabase.com](https://supabase.com) and create a new project.
2. In the Supabase dashboard → **SQL Editor** → **New Query**.
3. Paste and run the contents of `supabase_schema.sql`.
4. This creates 4 tables: `users`, `barcode_master`, `inward_headers`, `inward_line_items`.
5. Copy your **Project URL** and **anon/service key** from **Settings → API**.

### Step 2 — Local Setup

```bash
# Clone the repo
git clone https://github.com/YOUR_USERNAME/cdrsl-barcode-app.git
cd cdrsl-barcode-app

# Create virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Configure credentials
cp .streamlit/secrets.toml.template .streamlit/secrets.toml
# Edit secrets.toml and fill in SUPABASE_URL and SUPABASE_KEY
```

### Step 3 — Run Locally

```bash
streamlit run app.py
```

### Step 4 — Deploy to Streamlit Cloud

1. Push your code to GitHub (secrets.toml is git-ignored).
2. Go to [https://share.streamlit.io](https://share.streamlit.io).
3. Click **New app** → select your repo → `app.py`.
4. Under **Advanced settings → Secrets**, paste:
   ```toml
   SUPABASE_URL = "https://xxx.supabase.co"
   SUPABASE_KEY = "your-anon-key"
   ```
5. Click **Deploy**.

---

## 📱 Application Modules

| Module | Description |
|---|---|
| 🏠 Dashboard | Live stats: sessions, items scanned, users, master records |
| 👤 User Setup | Add/view verifiers & operators |
| 📂 Master Upload | Upload barcode master from Excel/CSV |
| 📋 New Inward Session | Enter header details, auto-generate document no |
| 🔍 Scan & Capture | Barcode scan, expiry entry, shelf life calculation |
| 📊 Reports | View & download Excel report per session |

---

## 🔢 Shelf Life Formula

Derived from the provided `Shelf_Life.xlsx`:

```
Total Days   = Expiry Date − Manufacturing Date
Remaining    = Expiry Date − Inward Date
Shelf Life % = (Remaining / Total Days) × 100
```

Items below the configured **Expiry Threshold %** are flagged in red on both the UI and the Excel report.

---

## 🗄️ Database Tables

### `users`
| Column | Type | Notes |
|---|---|---|
| id | uuid | PK |
| name | text | Full name |
| email | text | Unique |
| role | text | Verifier / Supervisor / Admin |
| department | text | |
| google_id | text | Google account / employee ID |
| is_active | boolean | |

### `barcode_master`
| Column | Type | Notes |
|---|---|---|
| id | uuid | PK |
| item_no | text | |
| barcode | text | Unique — upsert key |
| description | text | |
| uom | text | Optional |
| category | text | Optional |

### `inward_headers`
| Column | Type | Notes |
|---|---|---|
| document_no | text | Unique — CDRSL/BARCODE/001 |
| invoice_no | text | |
| boe_no | text | Bill of Entry No |
| boe_date | date | |
| inward_date | date | Inward at Port |
| goods_receipt_date | date | Receipt at Warehouse |
| file_no | text | |
| container_no | text | |
| invoice_lines | integer | |
| actual_lines | integer | |
| expiry_required | text | Yes / No |
| expiry_threshold | numeric | % |
| verified_by | text | |
| status | text | Active / Completed / Cancelled |

### `inward_line_items`
| Column | Type | Notes |
|---|---|---|
| document_no | text | FK → inward_headers |
| serial_no | text | Auto-generated |
| item_no | text | |
| barcode | text | |
| description | text | |
| remark | text | "Already Barcode Exists" / "New Item" |
| mfg_date | date | |
| expiry_date | date | |
| shelf_life_pct | numeric | Calculated |
| qty | integer | |
| verified_by | text | |

---

## 📊 Excel Report Layout

**Header Block:**
```
Document No | Date & Time | Inward Date | Expiry Threshold %
Invoice No  | File No     | Bill of Entry No
BOE Date    | Container No | Status
Verified By
```

**Line Items:**
```
Sl.No | Item No | Item Description | Barcode | Expiry Date | Mfg Date | Shelf Life % | Qty | Remarks | Verified By
```

- Items below threshold highlighted in **red**
- New items highlighted in **green**

---

## ⚙️ Key Business Rules

1. **Document Number Series**: Starts at `CDRSL/BARCODE/001`, auto-increments.
2. **One item → multiple barcodes**: Supported via multiple scan entries.
3. **One item → multiple expiry**: Supported via "No. of Expiry Entries" control.
4. **New items**: Added to barcode master automatically on scan.
5. **Session cancel**: Updates header status to `Cancelled`, stops further scanning.
6. **Scan limit**: Session automatically completes when scanned = actual lines.

---

## 👤 Author

**Created by Sweetson Joseph**  
CDRSL Warehouse Management · 2025

---

## 📄 License

Internal use — CDRSL. All rights reserved.
