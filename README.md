# Bang For Your Buck 🍺💶

**Find the best value for everyday essentials across Ireland**

Bang For Your Buck is a crowd-sourced price tracking platform helping Irish consumers find the best deals on everyday purchases. Starting with pint prices (because priorities!), we're building Ireland's most comprehensive value-finding community.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![Django 5.0+](https://img.shields.io/badge/django-5.0+-green.svg)](https://www.djangoproject.com/)
[![React 18+](https://img.shields.io/badge/react-18+-61dafb.svg)](https://reactjs.org/)

##  Vision

In a time of rising costs, Irish consumers deserve transparency, and it's not going to come from our government :)) . Bang For Your Buck empowers people to:
- **Find the best value** near them in real-time
- **Make informed decisions** with crowd-sourced data
- **Hold businesses accountable** through price transparency
- **Build community** around smart spending

##  Current Features (Phase 1: Pints)

### For Users
-  **Interactive Map** - Find pubs near you with color-coded price indicators
-  **Smart Search** - Search by venue name, location, or drink type
-  **Price Tracking** - See current prices and 30-day trends
-  **Statistics** - County averages, cheapest/most expensive pints
-  **Mobile First** - Optimized for on-the-go submissions
-  **Photo Upload** - Snap your receipt for verification
-  **Community Voting** - Upvote accurate prices, downvote errors

### For Contributors
-  **Quick Submit** - Report prices in under 30 seconds
-  **GPS Auto-detect** - Automatically finds nearby venues
-  **Leaderboards** - See top contributors (coming soon)
-  **Badges** - Earn rewards for contributions (coming soon)

## 📋 Roadmap

### Phase 1: Pints (Q4 2024)
- [x] MVP Planning
- [ ] Core platform development
- [ ] Launch with Dublin, Cork, Galway coverage
- [ ] 1,000 price submissions target

### Phase 2: Community Features (Q1 2025)
- [ ] User accounts & profiles
- [ ] Badges & gamification
- [ ] Price alerts & notifications
- [ ] Pub ratings & reviews

### Phase 3: Expansion (Q2 2025)
New categories:
- ☕ **Coffee** - Track your morning brew prices
- 🥪 **Chicken Fillet Rolls** - Ireland's favorite lunch
- 🍕 **Takeaway** - Pizzas, chipper prices, meal deals
- 🥖 **Supermarket Staples** - Bread, milk, eggs
- ⛽ **Fuel Prices** - Real-time petrol/diesel tracking
- 🎬 **Cinema Tickets** - Movie prices & concessions
- 💇 **Haircuts** - Barbers & salons
- 🏋️ **Gym Memberships** - Monthly costs across chains

### Phase 4: Intelligence (Q3 2025)
- [ ] Price prediction algorithms
- [ ] Personalized recommendations
- [ ] Business analytics dashboard
- [ ] API for third-party integrations

## 🏗️ Tech Stack

### Backend
- **Framework:** Django 5.0+ with Django REST Framework
- **Database:** PostgreSQL 14+ with PostGIS (geospatial queries)
- **Caching:** Redis 7+
- **Task Queue:** Celery with Redis broker
- **Search:** PostgreSQL Full-Text Search
- **Image Storage:** Cloudinary
- **Authentication:** JWT with dj-rest-auth (Phase 2)

### Frontend
- **Framework:** React 18+ with TypeScript
- **State Management:** TanStack Query (React Query) + Zustand
- **Routing:** React Router v6
- **Styling:** TailwindCSS 3+
- **Maps:** Google Maps JavaScript API / Mapbox GL JS
- **Charts:** Chart.js / Recharts
- **Forms:** React Hook Form + Zod validation
- **Build Tool:** Vite

### Infrastructure
- **Backend Hosting:** Railway / Render / DigitalOcean
- **Frontend Hosting:** Vercel / Netlify
- **CDN:** Cloudflare
- **Monitoring:** Sentry (errors) + UptimeRobot (uptime)
- **Analytics:** Plausible / Google Analytics
- **CI/CD:** GitHub Actions

## 🛠️ Development Setup

### Prerequisites
```bash
# Required
- Python 3.10+
- Node.js 18+
- PostgreSQL 14+ with PostGIS
- Redis 7+

# Optional but recommended
- Docker & Docker Compose (for easy setup)
```

### Quick Start with Docker (Recommended)
```bash
# Clone the repository
git clone https://github.com/yourusername/bangforyourbuck.git
cd bangforyourbuck

# Copy environment files
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env

# Edit .env files with your API keys (see Configuration section)

# Start all services
docker-compose up -d

# Run migrations
docker-compose exec backend python manage.py migrate

# Create superuser
docker-compose exec backend python manage.py createsuperuser

# Load seed data
docker-compose exec backend python manage.py loaddata seed_data

# Access the app
# Frontend: http://localhost:3000
# Backend API: http://localhost:8000/api/
# Django Admin: http://localhost:8000/admin/
```

### Manual Setup

#### Backend Setup
```bash
# Navigate to backend directory
cd backend

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Set up PostgreSQL database
createdb bangforyourbuck
psql bangforyourbuck -c "CREATE EXTENSION postgis;"

# Configure environment variables
cp .env.example .env
# Edit .env with your settings

# Run migrations
python manage.py migrate

# Create superuser
python manage.py createsuperuser

# Load seed data (drink types, sample venues)
python manage.py loaddata fixtures/drink_types.json
python manage.py loaddata fixtures/sample_venues.json

# Start development server
python manage.py runserver
```

#### Frontend Setup
```bash
# Navigate to frontend directory (in a new terminal)
cd frontend

# Install dependencies
npm install

# Configure environment variables
cp .env.example .env.local
# Edit .env.local with your API keys

# Start development server
npm run dev
```

#### Start Background Workers
```bash
# In another terminal, start Celery worker
cd backend
celery -A config worker -l info

# Optional: Start Celery beat for scheduled tasks
celery -A config beat -l info
```

## ⚙️ Configuration

### Required API Keys

#### Google Maps (or Mapbox alternative)
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Enable these APIs:
   - Maps JavaScript API
   - Geocoding API
   - Places API (optional, for venue discovery)
3. Create API key and restrict to your domains
4. Add to `frontend/.env.local`:
```env
VITE_GOOGLE_MAPS_API_KEY=your_key_here
```

#### Cloudinary (Image Hosting)
1. Sign up at [Cloudinary](https://cloudinary.com/)
2. Get your credentials from Dashboard
3. Add to `backend/.env`:
```env
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret
```

#### Sentry (Error Tracking - Optional)
1. Sign up at [Sentry.io](https://sentry.io/)
2. Create project
3. Add DSN to both `.env` files:
```env
# Backend
SENTRY_DSN=your_backend_dsn

# Frontend
VITE_SENTRY_DSN=your_frontend_dsn
```

### Environment Variables

#### Backend (.env)
```env
# Django
SECRET_KEY=your-secret-key-here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/bangforyourbuck

# Redis
REDIS_URL=redis://localhost:6379/0

# Cloudinary
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# Email (for notifications - Phase 2)
EMAIL_BACKEND=django.core.mail.backends.console.EmailBackend
EMAIL_HOST=smtp.sendgrid.net
EMAIL_PORT=587
EMAIL_USE_TLS=True
EMAIL_HOST_USER=apikey
EMAIL_HOST_PASSWORD=your_sendgrid_api_key

# Sentry
SENTRY_DSN=your_sentry_dsn

# CORS
CORS_ALLOWED_ORIGINS=http://localhost:3000,http://127.0.0.1:3000
```

#### Frontend (.env.local)
```env
# API
VITE_API_BASE_URL=http://localhost:8000/api

# Google Maps
VITE_GOOGLE_MAPS_API_KEY=your_google_maps_key

# Sentry
VITE_SENTRY_DSN=your_sentry_dsn

# Environment
VITE_ENVIRONMENT=development
```

## 📁 Project Structure
```
bangforyourbuck/
├── backend/                  # Django backend
│   ├── config/              # Project settings
│   ├── apps/
│   │   ├── venues/          # Venue management
│   │   ├── prices/          # Price submissions
│   │   ├── users/           # User management (Phase 2)
│   │   └── analytics/       # Statistics & reporting
│   ├── utils/               # Shared utilities
│   ├── fixtures/            # Seed data
│   ├── requirements.txt
│   └── manage.py
│
├── frontend/                # React frontend
│   ├── src/
│   │   ├── components/      # Reusable components
│   │   │   ├── common/      # Buttons, inputs, etc.
│   │   │   ├── map/         # Map-related components
│   │   │   ├── forms/       # Form components
│   │   │   └── layout/      # Navigation, footer
│   │   ├── pages/           # Page components
│   │   ├── hooks/           # Custom React hooks
│   │   ├── services/        # API service layer
│   │   ├── utils/           # Helper functions
│   │   ├── types/           # TypeScript types
│   │   ├── App.tsx
│   │   └── main.tsx
│   ├── public/              # Static assets
│   ├── package.json
│   └── vite.config.ts
│
├── docker-compose.yml       # Docker configuration
├── .github/
│   └── workflows/           # CI/CD pipelines
│       ├── backend-tests.yml
│       └── frontend-tests.yml
├── docs/                    # Additional documentation
│   ├── API.md              # API documentation
│   ├── CONTRIBUTING.md     # Contribution guidelines
│   └── DEPLOYMENT.md       # Deployment guide
├── README.md               # This file
└── LICENSE                 # MIT License
```

## 🧪 Testing

### Backend Tests
```bash
cd backend

# Run all tests
python manage.py test

# Run with coverage
coverage run --source='.' manage.py test
coverage report
coverage html  # Generate HTML report

# Run specific app tests
python manage.py test apps.venues
python manage.py test apps.prices

# Lint code
flake8 .
black . --check
isort . --check-only
```

### Frontend Tests
```bash
cd frontend

# Run unit tests
npm test

# Run with coverage
npm test -- --coverage

# Run e2e tests (when implemented)
npm run test:e2e

# Lint code
npm run lint

# Type checking
npm run type-check
```

## 📊 Database Models

### Core Models

#### Venue
```python
- id (UUID, PK)
- name (string)
- address_line_1 (string)
- city (string)
- county (choice: 32 Irish counties)
- eircode (string, optional)
- latitude (decimal)
- longitude (decimal)
- venue_type (choice: pub, hotel_bar, nightclub, restaurant)
- phone (string, optional)
- website (URL, optional)
- image (ImageField)
- is_verified (boolean)
- is_active (boolean)
- created_at (datetime)
- updated_at (datetime)
```

#### Category
```python
- id (Auto, PK)
- name (string: "Pints", "Coffee", "Food", etc.)
- slug (string)
- icon (string: icon identifier)
- is_active (boolean)
- order (integer: display order)
```

#### DrinkType (extends base Item model)
```python
- id (Auto, PK)
- category (FK: Category)
- name (string: "Guinness", "Heineken", etc.)
- item_type (choice: beer, cider, wine, spirits)
- is_active (boolean)
```

#### PriceSubmission
```python
- id (UUID, PK)
- venue (FK: Venue)
- item (FK: DrinkType/FoodItem/etc.)
- price (decimal)
- submitted_at (datetime)
- photo (ImageField, optional)
- comment (text, optional, max 280)
- ip_address (IP)
- user_agent (string)
- latitude (decimal, optional: submission location)
- longitude (decimal, optional)
- upvotes (integer)
- downvotes (integer)
- is_verified (boolean)
- is_flagged (boolean)
- user (FK: User, optional - Phase 2)
```

#### Vote
```python
- id (Auto, PK)
- submission (FK: PriceSubmission)
- ip_address (IP)
- vote_type (choice: 'up', 'down')
- voted_at (datetime)
```

#### Report
```python
- id (Auto, PK)
- submission (FK: PriceSubmission, optional)
- venue (FK: Venue, optional)
- report_type (choice: incorrect_price, closed, wrong_location, spam)
- description (text)
- ip_address (IP)
- created_at (datetime)
- status (choice: pending, reviewed, resolved)
- admin_notes (text, optional)
```

## 🔌 API Endpoints

### Venues
```
GET    /api/venues/                  # List venues (with filtering)
GET    /api/venues/{id}/             # Venue detail
GET    /api/venues/{id}/prices/      # Prices for specific venue
GET    /api/venues/nearby/           # Venues within radius
POST   /api/venues/                  # Create venue (admin/verified users)
```

### Prices
```
GET    /api/prices/                  # List prices (with filtering)
POST   /api/prices/                  # Submit new price
GET    /api/prices/{id}/             # Price detail
POST   /api/prices/{id}/vote/        # Vote on price accuracy
```

### Categories & Items
```
GET    /api/categories/              # List categories
GET    /api/items/                   # List items (drinks, food, etc.)
GET    /api/items/?category={slug}   # Filter items by category
```

### Statistics
```
GET    /api/stats/                   # Overall statistics
GET    /api/stats/counties/          # County-level averages
GET    /api/stats/weekly/            # Weekly highlights
```

### Reports
```
POST   /api/reports/                 # Submit report
```

See [API.md](docs/API.md) for complete API documentation.

## 🎨 Design System

### Color Palette
```css
/* Price Indicators */
--price-cheap: #22c55e     /* Green - Good value */
--price-medium: #eab308    /* Yellow - Average */
--price-expensive: #ef4444 /* Red - Pricey */

/* Brand Colors */
--primary: #2563eb         /* Blue */
--secondary: #7c3aed       /* Purple */
--accent: #f59e0b          /* Amber */

/* Neutrals */
--background: #ffffff
--surface: #f9fafb
--border: #e5e7eb
--text-primary: #111827
--text-secondary: #6b7280
```

### Typography
- **Font Family:** Inter (primary), System UI (fallback)
- **Headings:** Bold, 1.5rem–3rem
- **Body:** Regular, 1rem
- **Small:** 0.875rem

### Spacing Scale (Tailwind)
- xs: 0.25rem (4px)
- sm: 0.5rem (8px)
- md: 1rem (16px)
- lg: 1.5rem (24px)
- xl: 2rem (32px)

## 🤝 Contributing

We love contributions! Whether you're fixing bugs, adding features, or improving docs, we want your help.

### How to Contribute

1. **Fork the repository**
2. **Create a feature branch**
```bash
   git checkout -b feature/your-feature-name
```
3. **Make your changes**
   - Write tests for new features
   - Follow existing code style
   - Update documentation
4. **Commit with clear messages**
```bash
   git commit -m "feat: add price alert notifications"
```
5. **Push to your fork**
```bash
   git push origin feature/your-feature-name
```
6. **Open a Pull Request**

### Commit Convention
We follow [Conventional Commits](https://www.conventionalcommits.org/):
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `style:` Code style changes (formatting)
- `refactor:` Code refactoring
- `test:` Adding/updating tests
- `chore:` Maintenance tasks

### Code Style
- **Python:** Follow PEP 8, use Black for formatting
- **JavaScript/TypeScript:** Follow Airbnb style guide, use Prettier
- **Maximum line length:** 88 characters (Python), 100 (JS/TS)

### Testing Requirements
- All new features must include tests
- Maintain >80% code coverage
- All tests must pass before merging

See [CONTRIBUTING.md](docs/CONTRIBUTING.md) for detailed guidelines.

## 🚀 Deployment

### Production Deployment

#### Backend (Railway/Render)
```bash
# Configure environment variables in dashboard
# Deploy via GitHub integration or CLI

railway up  # Railway
# or
render deploy  # Render
```

#### Frontend (Vercel/Netlify)
```bash
# Configure environment variables in dashboard
# Deploy via GitHub integration or CLI

vercel --prod  # Vercel
# or
netlify deploy --prod  # Netlify
```

See [DEPLOYMENT.md](docs/DEPLOYMENT.md) for detailed deployment guide.

## 📈 Success Metrics

### Launch Targets (Month 1)
- ✅ 300 venues seeded
- 🎯 1,000 price submissions
- 🎯 5,000 unique visitors
- 🎯 Media coverage in 2+ publications

### 6-Month Goals
- 🎯 2,000 venues
- 🎯 20,000 price submissions
- 🎯 50,000 unique visitors
- 🎯 100 organic submissions per day

### 12-Month Vision
- 🎯 5,000 venues (comprehensive coverage)
- 🎯 100,000 price submissions
- 🎯 200,000 unique visitors
- 🎯 3+ categories launched
- 🎯 Self-sustaining community

## 🔐 Security

- All API endpoints rate-limited
- User inputs sanitized
- HTTPS enforced in production
- CORS properly configured
- Environment variables never committed
- Regular dependency updates
- SQL injection protection (Django ORM)
- XSS protection enabled

### Reporting Security Issues
Please email security@bangforyourbuck.ie (do not open public issues)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 💬 Community & Support

- **Website:** [bangforyourbuck.ie](https://bangforyourbuck.ie)
- **Issues:** [GitHub Issues](https://github.com/kimatron/bangforyourbuck/issues)
- **Discussions:** [GitHub Discussions](https://github.com/kimatron/bangforyourbuck/discussions)
- **Twitter:** [@bangforbuckIE](https://twitter.com/bangforbuckIE)
- **Email:** hello@bangforyourbuck.ie

## 🙏 Acknowledgments

- Built with ❤️ by the Irish developer community
- Inspired by the rising cost of living in Ireland
- Thanks to all contributors and price reporters
- Map data © OpenStreetMap contributors
- Venue data sourced from public directories

## 🗺️ Roadmap to Success

### Week 1-2: Foundation
- [x] Project planning
- [ ] Repository setup
- [ ] Database design
- [ ] Basic Django models
- [ ] React project scaffolding

### Week 3-4: Core Features
- [ ] Venue CRUD operations
- [ ] Price submission API
- [ ] Map view with pins
- [ ] Search functionality
- [ ] Basic filtering

### Week 5-6: Polish & Data
- [ ] Mobile responsiveness
- [ ] Seed 300 Dublin/Cork/Galway venues
- [ ] Statistics dashboard
- [ ] Photo upload
- [ ] Admin moderation panel

### Week 7-8: Launch Prep
- [ ] Production deployment
- [ ] Domain & SSL setup
- [ ] Error tracking
- [ ] Analytics
- [ ] Landing page copy
- [ ] Social media setup

### Week 9: Soft Launch
- [ ] Post to r/ireland
- [ ] Share on Twitter/X
- [ ] Email Irish tech bloggers
- [ ] Ask friends to submit prices
- [ ] Monitor feedback

### Week 10+: Growth
- [ ] Media outreach
- [ ] SEO optimization
- [ ] Community building
- [ ] Feature iteration based on feedback

---

** Prepping for the recession one user input at a time **

**Star ⭐ this repo if you find it useful!**

