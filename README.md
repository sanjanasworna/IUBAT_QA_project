# IUBAT Q&A Platform

A community Q&A platform for IUBAT students to ask questions and get answers from verified peers.

![Status](https://img.shields.io/badge/status-active-success)
![Django](https://img.shields.io/badge/Django-5.x-092E20?logo=django)
![Next.js](https://img.shields.io/badge/Next.js-15.x-000000?logo=next.js)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-14+-336791?logo=postgresql)

---

## Tech Stack

| Layer    | Technologies                              |
|----------|-------------------------------------------|
| Frontend | Next.js 15, Tailwind CSS v3, Axios        |
| Backend  | Django 5, Django REST Framework, JWT      |
| Database | PostgreSQL                                |
| Hosting  | Vercel (frontend), Render (backend, db)   |

---

## Features

- ✔ JWT authentication (register, login, logout)
- ✔ Student ID verification with image upload
- ✔ Ask questions with tags
- ✔ Post answers (verified users only)
- ✔ Upvote questions and answers (toggle)
- ✔ Search and filter by tag
- ✔ User profile with edit
- ✔ Django admin panel

---

## Local Setup

### Prerequisites

- Python 3.12+
- Node.js 18+ LTS
- PostgreSQL 14+

---

### 1. Clone

```bash
git clone https://github.com/YOUR_USERNAME/iubat-qa-platform.git
cd iubat-qa-platform
```

---

### 2. Database

Open pgAdmin or psql and run:

```sql
CREATE DATABASE iubat_qa_db;
```

---

### 3. Backend

```bash
cd backend
```

**Create and activate virtual environment:**

On Windows:
```bash
python -m venv .venv
.venv\Scripts\activate
```

On Mac/Linux:
```bash
python3 -m venv .venv
source .venv/bin/activate
```

**Install dependencies:**
```bash
pip install -r requirements.txt
```

**Create `backend/.env` file:**

```env
SECRET_KEY=your-secret-key
DEBUG=True
DB_NAME=iubat_qa_db
DB_USER=postgres
DB_PASSWORD=your-password
DB_HOST=localhost
DB_PORT=5432
```

**Generate a secret key:**

```bash
python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"
```

**Run migrations and start the server:**

```bash
python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

Backend will be available at → `http://127.0.0.1:8000`

---

### 4. Seed Tags

Open the Django shell:

```bash
python manage.py shell
```

Then run the following inside the shell:

```python
from questions.models import Tag

tags = [
    {"name": "Admissions",     "slug": "admissions"},
    {"name": "CSE Department", "slug": "cse-department"},
    {"name": "Campus Life",    "slug": "campus-life"},
    {"name": "Academic",       "slug": "academic"},
    {"name": "Faculty",        "slug": "faculty"},
    {"name": "Clubs & Events", "slug": "clubs-events"},
    {"name": "Transport",      "slug": "transport"},
    {"name": "Hostel",         "slug": "hostel"},
    {"name": "General",        "slug": "general"},
]

for tag in tags:
    Tag.objects.get_or_create(name=tag["name"], slug=tag["slug"])

print("Done!")
exit()
```

---

### 5. Frontend

```bash
cd frontend
npm install
npm run dev
```

- Frontend → `http://localhost:3000`
- Admin Panel → `http://127.0.0.1:8000/admin`

---

## API Reference

| Method | Endpoint                        | Description              | Auth          |
|--------|---------------------------------|--------------------------|---------------|
| POST   | `/api/users/register/`          | Register                 | No            |
| POST   | `/api/users/login/`             | Login, returns JWT       | No            |
| GET    | `/api/users/profile/`           | Get profile              | Yes           |
| PUT    | `/api/users/profile/`           | Update profile           | Yes           |
| GET    | `/api/users/verify/`            | Check verify status      | Yes           |
| POST   | `/api/users/verify/`            | Submit ID image          | Yes           |
| GET    | `/api/questions/`               | List, search, filter     | No            |
| POST   | `/api/questions/`               | Create question          | Yes           |
| GET    | `/api/questions/<id>/`          | Question detail          | No            |
| GET    | `/api/questions/tags/`          | All tags                 | No            |
| POST   | `/api/answers/questions/<id>/`  | Post answer              | Verified only |
| POST   | `/api/votes/questions/<id>/`    | Toggle question vote     | Yes           |
| POST   | `/api/votes/answers/<id>/`      | Toggle answer vote       | Yes           |

---

## Deployment

### Frontend → Vercel

1. Import the repo on [vercel.com](https://vercel.com)
2. Set root directory to `frontend`
3. Add the following environment variable:
   ```
   NEXT_PUBLIC_API_URL = https://your-backend.onrender.com/api
   ```
4. Deploy

Update `frontend/src/lib/axios.js`:

```js
baseURL: process.env.NEXT_PUBLIC_API_URL
```

---

### Backend → Render

**Install production packages:**

```bash
pip install gunicorn whitenoise dj-database-url
pip freeze > requirements.txt
```

**Create `backend/build.sh`:**

```bash
#!/usr/bin/env bash
set -o errexit
pip install -r requirements.txt
python manage.py collectstatic --no-input
python manage.py migrate
```

**Render service settings:**

| Setting       | Value                                                              |
|---------------|--------------------------------------------------------------------|
| Environment   | Python                                                             |
| Build Command | `chmod +x build.sh && ./build.sh`                                  |
| Start Command | `gunicorn core.wsgi:application --workers 4 --bind 0.0.0.0:$PORT` |

**Render environment variables:**

| Key          | Value                        |
|--------------|------------------------------|
| SECRET_KEY   | your-secret-key              |
| DEBUG        | False                        |
| DATABASE_URL | (copied from Render PostgreSQL) |

**Update `core/settings.py` for production:**

```python
import dj_database_url

DEBUG         = os.environ.get('DEBUG', 'False') == 'True'
ALLOWED_HOSTS = ['.onrender.com', 'localhost']

DATABASE_URL = os.environ.get('DATABASE_URL')
if DATABASE_URL:
    DATABASES = {'default': dj_database_url.parse(DATABASE_URL)}

CORS_ALLOWED_ORIGINS = [
    'http://localhost:3000',
    'https://your-app.vercel.app',
]

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'whitenoise.middleware.WhiteNoiseMiddleware',
    # ...rest of middleware
]

STATIC_ROOT = BASE_DIR / 'staticfiles'
```

---

### Database → Render PostgreSQL

1. Go to Render dashboard → **New** → **PostgreSQL**
2. Copy the **External Database URL**
3. Paste it as the `DATABASE_URL` environment variable in your backend service

---

## Contributing

```bash
# 1. Fork the repo

# 2. Create a branch
git checkout -b feature/your-feature

# 3. Commit your changes
git commit -m "feat: your feature description"

# 4. Push and open a Pull Request
git push origin feature/your-feature
```

### Commit Convention

| Prefix     | Usage                              |
|------------|------------------------------------|
| `feat:`     | New feature                        |
| `fix:`      | Bug fix                            |
| `docs:`     | Documentation changes              |
| `refactor:` | Code change without feature change |

---

## Known Issues

- → Media files are lost on Render redeploy — **Fix:** Use Cloudinary
- → No email notifications yet — **Fix:** Celery + SendGrid
- → No pagination on questions feed yet

---

## License

MIT © 2024 IUBAT Q&A Platform