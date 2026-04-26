# EMRims SO Suite v1.0

**Sales Operations Suite for Medical Professionals**

Built for the Abuja Medical Corridor by impactBot Tech Solution.

## 🎨 Visual Architecture: "Zen Speed"

- **Primary Color**: EMR Orange (#FF8C00)
- **Data Viz Color**: Deep Medical Teal (#008080)
- **Background**: Matte White (#FDFDFD)
- **Interaction**: Zero-clutter dashboard with fade-out tooltips
- **Modal Style**: Scale transition (0.95→1.0) with 10px backdrop blur

## 📦 Tech Stack

- **Frontend**: React 18+ with TypeScript
- **Backend**: Node.js + Express.js
- **Database**: PostgreSQL
- **Authentication**: JWT + bcrypt
- **State Management**: Zustand + Redux Toolkit
- **Styling**: Tailwind CSS + Framer Motion
- **API**: RESTful with OpenAPI documentation

## 🚀 Core Modules

1. **Stealth Authentication** — Minimalist login, JWT persistence, 30-day session
2. **Smart Measurement Assistant** — AI motion capture, EMR sizing engine, WhatsApp PDF
3. **CRM & Lead Tracker** — Multi-channel follow-up, activity verification, Excel export
4. **Digital Portfolio** — Three fabric tiers, branded PDF generation
5. **Executive Analytics** — Performance radar, commission gauge, downloadable reports

## 📁 Project Structure

```
emrims-so-suite/
├── frontend/                 # React TypeScript application
├── backend/                  # Node.js Express server
├── database/                 # PostgreSQL migrations
├── docs/                     # Documentation
├── docker-compose.yml        # Local development setup
└── README.md
```

## 🛠 Quick Start

### Using Docker Compose (Recommended)
```bash
# Start all services (PostgreSQL, Backend, Frontend)
docker-compose up
```

### Manual Setup

**Frontend:**
```bash
cd frontend
npm install
npm run dev
```

**Backend:**
```bash
cd backend
npm install
npm run dev
```

**Database:**
```bash
cd backend
npm run migrate
npm run seed
```

## 🔐 Default Credentials

- **Username**: `admin`
- **Password**: `Admin`

## 📊 Access Points

- **Frontend**: http://localhost:3000
- **Backend API**: http://localhost:5000
- **PostgreSQL**: localhost:5432

## 📚 Documentation

- [Architecture Guide](./docs/ARCHITECTURE.md)
- [API Documentation](./docs/API.md)
- [Database Schema](./docs/DATABASE.md)

## 👨‍💼 Project Lead

**Developer**: abugaldis-tech  
**Brand**: impactBot Tech Solution  
**Territory**: Abuja Medical Corridor  
**Last Updated**: 2026-04-26

## 📄 License

Proprietary - impactBot Tech Solution