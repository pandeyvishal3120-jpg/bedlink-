# BedLink (backend + frontend in one)
pip install -r requirements.txt
python -m uvicorn main:app --reload --port 8000
Open http://localhost:8000 (app) | /docs (API tester). Data is saved in bedlink.db (auto-created). Reset: POST /api/reset
# 🚑 BedLink

### Smart Emergency Hospital Bed Coordination System

BedLink is a healthcare coordination platform designed to help ambulance crews identify nearby hospitals with the required emergency beds and specialties.

The system evaluates hospital availability, distance, estimated arrival time, bed freshness, and other factors to rank hospitals for an emergency request.

---

## 🎯 Problem

During a medical emergency, an ambulance may need to find a hospital that has the required resources **right now**, such as:

* ICU
* Ventilator
* Oxygen
* Cardiac
* Burns
* Trauma
* NICU
* Stroke

Calling multiple hospitals manually can waste valuable time.

**BedLink provides a centralized system for checking hospital resources and coordinating emergency requests.**

---

## 💡 Key Features

### 🏥 Hospital Bed Management

Hospitals can update their available beds for different emergency categories.

Supported bed types:

* ICU
* Ventilator
* Oxygen
* Cardiac
* Burns
* Trauma
* NICU
* Stroke

### 🚑 Smart Hospital Ranking

BedLink evaluates hospitals using factors including:

* Required bed availability
* Distance from the ambulance
* Estimated travel time
* Bed information freshness
* Hospital load
* Probability of bed availability at arrival
* Required-bed matching

The backend calculates a score and ranks hospitals accordingly.

### 📍 Distance & ETA

The system calculates approximate distance between the ambulance location and hospitals and estimates travel time.

The current demo uses a road-distance factor and a fixed demo speed of **30 km/h**.

### 🔄 Emergency Request Flow

An emergency request can be created with:

* Required beds/specialties
* Patient condition
* Ambulance latitude
* Ambulance longitude

The system then generates a ranked list and sends an offer to a suitable hospital.

### ⏱️ Hospital Offer & Timeout

Hospital offers have a **120-second response window**.

A hospital can:

* Accept the request
* Reject the request

If an offer times out, the system automatically moves to the next hospital.

### 🔐 Hospital Login

Hospitals can log in using a hospital ID and demo PIN.

For the demo:

```text
PIN = Hospital ID as 4 digits
```

Example:

```text
Hospital ID: 7
PIN: 0007
```

### 🗃️ SQLite Persistence

BedLink automatically creates a local SQLite database:

```text
bedlink.db
```

The database stores:

* Hospitals
* Bed inventory
* Bed update history
* Emergency requests

### 📊 Bed Update History

The system records bed inventory changes with:

* Hospital ID
* Bed type
* Previous count
* New count
* Timestamp
* Update source

### 🚨 Ambulance Simulation

After a hospital accepts a request, the backend simulates ambulance movement toward the selected hospital for the demo.

---

## 🏗️ Architecture

```text
                    ┌──────────────────────┐
                    │      BedLink UI      │
                    │     static/ folder   │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │      FastAPI API     │
                    │       main.py        │
                    └──────────┬───────────┘
                               │
              ┌────────────────┼────────────────┐
              │                │                │
              ▼                ▼                ▼
        Hospital Beds     Ranking Engine    Request System
              │                │                │
              └────────────────┼────────────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │       SQLite         │
                    │     bedlink.db       │
                    └──────────────────────┘
```

---

## 🛠️ Technology Stack

### Backend

* Python
* FastAPI
* Uvicorn
* Pydantic

### Database

* SQLite

### Frontend

* Static frontend served through FastAPI
* Files located inside the `static/` directory

### API Documentation

FastAPI automatically provides interactive API documentation through Swagger UI.

---

## 📁 Project Structure

```text
BedLink/
│
├── main.py
├── requirements.txt
├── README.md
├── bedlink.db
│
└── static/
    ├── index.html
    └── frontend files...
```

> `bedlink.db` is automatically created when the application initializes the database.

---

# 🚀 Installation & Setup

## 1. Clone the repository

```bash
git clone https://github.com/YOUR-USERNAME/BedLink.git
cd BedLink
```

Replace `YOUR-USERNAME` with your GitHub username.

---

## 2. Create a virtual environment

### Windows

```powershell
python -m venv venv
```

Activate it:

```powershell
venv\Scripts\activate
```

### macOS / Linux

```bash
python3 -m venv venv
source venv/bin/activate
```

---

## 3. Install dependencies

```bash
pip install -r requirements.txt
```

---

## 4. Run the application

```bash
python -m uvicorn main:app --reload --port 8000
```

The server will start at:

```text
http://127.0.0.1:8000
```

---

# 🌐 Open the Application

Open your browser and visit:

```text
http://localhost:8000
```

The frontend is served directly by the FastAPI application.

---

# 📚 API Documentation

FastAPI provides an interactive API testing interface.

Open:

```text
http://localhost:8000/docs
```

You can test endpoints directly from the Swagger interface.

---

# 🔌 Main API Endpoints

## Hospital APIs

### Get hospitals

```http
GET /api/hospitals
```

Returns hospital information and current bed availability.

### Update beds

```http
POST /api/beds/{hospital_id}
```

Updates the availability of a particular bed type.

### Confirm bed information

```http
POST /api/beds/{hospital_id}/confirm
```

Updates the freshness timestamp of the hospital's bed information.

---

## Ranking API

### Rank hospitals

```http
POST /api/rank
```

The request contains:

```json
{
  "needs": ["ICU", "Ventilator"],
  "lat": 19.0760,
  "lng": 72.8777
}
```

The API returns ranked hospitals with information such as:

* Distance
* ETA
* Bed match
* Availability at arrival
* Risk
* Hospital load
* Ranking score

---

## Emergency Request APIs

### Create emergency request

```http
POST /api/requests
```

### Get all requests

```http
GET /api/requests
```

### Get a specific request

```http
GET /api/requests/{request_id}
```

### Respond to request

```http
POST /api/requests/{request_id}/respond
```

A hospital can accept or reject an emergency request.

---

## Hospital Offers

```http
GET /api/hospitals/{hospital_id}/offers
```

Returns emergency offers associated with a hospital.

---

## Authentication

### Hospital login

```http
POST /api/login
```

Example:

```json
{
  "hospital_id": 7,
  "pin": "0007"
}
```

The backend returns an authentication token for subsequent hospital-specific operations.

---

# 🧮 Hospital Ranking Logic

BedLink calculates a ranking score using several factors.

The current implementation considers:

```text
35% → Probability of required beds being available
25% → Travel-time factor
20% → Required-bed matching
10% → Bed information freshness
10% → Hospital load
```

The system also calculates:

* Bed confidence
* Effective available beds
* Probability of availability at arrival
* Match percentage
* Risk level
* Distance
* ETA

The purpose is to provide a transparent ranking mechanism rather than simply selecting the geographically nearest hospital.

---

# 🔄 Emergency Workflow

```text
Ambulance
    │
    ▼
Enter Patient Requirements
    │
    ▼
Send Emergency Request
    │
    ▼
BedLink Evaluates Hospitals
    │
    ├── Bed Availability
    ├── Distance
    ├── ETA
    ├── Freshness
    └── Hospital Load
    │
    ▼
Rank Hospitals
    │
    ▼
Send Offer to Hospital
    │
    ├── Accept
    │     │
    │     ▼
    │   Bed Held
    │     │
    │     ▼
    │   Ambulance Simulation
    │
    └── Reject / Timeout
          │
          ▼
      Offer Next Hospital
```

---

# 🧪 Demo Mode

The backend includes a demo endpoint that can simulate hospital responses.

```http
POST /api/demo/respond/{request_id}
```

It can simulate:

* Hospital acceptance
* Hospital rejection
* Offer timeout

When an emergency request is accepted through the demo flow, the ambulance journey can be accelerated for demonstration purposes.

---

# 🔄 Reset Demo Data

The application provides a reset endpoint:

```http
POST /api/reset
```

This recreates the seeded hospital and bed data.

> Use this endpoint when you want to reset the demo environment.

---

# 🏥 Sample Hospitals

The demo dataset contains hospitals from Mumbai, including hospitals in areas such as:

* Parel
* Bandra
* Mahim
* Andheri
* Vile Parle
* Sion
* Mumbai Central
* Byculla
* Powai
* Borivali
* Ghatkopar
* Mulund

The hospital coordinates and initial bed counts are seeded by the backend for demonstration purposes.

---

# ⚠️ Current Demo Limitations

This project is currently designed as a prototype/demo.

The current implementation uses:

* Seeded hospital data
* Approximate demo coordinates
* Approximate ETA calculation
* Local SQLite database
* Demo authentication
* Simulated ambulance movement
* In-memory request state loaded from SQLite

The backend itself notes that the demo can later replace the current hospital/request storage with **PostgreSQL + Redis**.

---

# 🔮 Future Improvements

Possible future improvements include:

* Real hospital database
* PostgreSQL deployment
* Redis for real-time state
* Real GPS tracking
* Real road routing and traffic-based ETA
* Hospital verification
* Production authentication
* Role-based access control
* Push notifications
* WebSocket-based live updates
* Cloud deployment
* Analytics dashboard
* Real hospital integrations

---

# 📌 Project Status

**Prototype / Hackathon Project**

BedLink demonstrates how emergency hospital-bed coordination can be digitized through real-time bed updates, hospital ranking, emergency requests, and hospital acceptance workflows.

---

## 👨‍💻 Getting Started

After starting the server:

```bash
python -m uvicorn main:app --reload --port 8000
```

Open:

**Application**

```text
http://localhost:8000
```

**API Documentation**

```text
http://localhost:8000/docs
```

---

## 📄 License

This project is currently intended for educational, demonstration, and hackathon purposes.
