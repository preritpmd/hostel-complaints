# Hostel Complaints

A full-stack web app for reporting and tracking hostel maintenance issues. Students log in, file a complaint (with up to 3 photos), and follow its progress. Hostel secretaries manage every complaint from a live dashboard.

All data lives on the server in one central database. Nothing is stored in the browser.

**Live demo:** https://hostel-complaints-14iz.onrender.com
**Secretary dashboard:** `/admin` on the same URL

## Features

**Students**
- Register with an institute email (`rollno@iitdh.ac.in`), WhatsApp number, hostel, wing, floor and room
- File complaints by category (Water, Wi-Fi, Electrical, Cleanliness, Furniture, Other) with a description and up to 3 photos
- Get a ticket number per hostel, like `H2-0001`
- See the status, priority, assignee and remarks for each complaint
- Mark a complaint as resolved, or reopen it if the problem comes back

**Secretaries**
- Dashboard of all complaints for their hostel (or both hostels, depending on the account)
- Update status (Pending, In Progress, Rejected), priority (Low to Urgent), assignee and remarks
- A reason is required when rejecting a complaint
- The dashboard refreshes by itself every 8 seconds and shows when a new complaint arrives

## Tech stack

| Layer | Technology |
|---|---|
| Backend | Python, Flask, Gunicorn |
| Database | PostgreSQL in the cloud (SQLite for local testing), via SQLAlchemy |
| Frontend | HTML, CSS, vanilla JavaScript |
| Hosting | Render (web service + managed Postgres) |

## Project structure

```
.
├── app.py              # Flask app: API, auth, database models
├── static/             # Frontend served by Flask
│   ├── student.html    # Student page (served at /)
│   ├── admin.html      # Secretary page (served at /admin)
│   ├── student.js
│   ├── admin.js
│   ├── shared.js       # API helper functions used by both pages
│   ├── shared.css
│   ├── config.js       # API base URL setting
│   └── photos.js       # Embedded council photos
├── requirements.txt
├── Procfile            # Process definition for Heroku-style hosts
├── render.yaml         # Render blueprint (web service + database)
└── .env.example        # Environment variables to set in the cloud
```

## Run locally

```bash
python -m venv venv
venv\Scripts\activate          # Mac/Linux: source venv/bin/activate
pip install -r requirements.txt
python app.py
```

- Students: http://127.0.0.1:5000
- Secretary: http://127.0.0.1:5000/admin

Local mode uses a SQLite file (`hostel.db`) and creates a test secretary account, `sec@h1.edu` with password `secretary123`, which can see both hostels. **This account exists only in local mode** and is never created in the cloud. Local mode is for testing only.

If you change the database tables between versions, delete `hostel.db` and restart.

## Deploy to the cloud (Render)

1. Push this repository to GitHub.
2. In Render, choose **New → Blueprint** and select the repo. Render reads `render.yaml` and creates the web service and a Postgres database.
3. When asked for `ADMINS`, enter the secretary accounts (format below).
4. Wait for the service to show **Live**, then open its URL.

Any host that runs Python and Postgres works the same way (Railway, Fly.io, Heroku, a VPS). Set `DATABASE_URL`, `SECRET_KEY` and `ADMINS`, then run `gunicorn app:app`.

Note: Render's free plan puts the service to sleep when idle (the first request can take 50 seconds or more), and free Postgres databases may expire. Check Render's current terms, and use a paid database for real long-term use.

## Environment variables

| Variable | Required | Description |
|---|---|---|
| `DATABASE_URL` | In the cloud | Postgres connection string. Setting it switches the app to cloud mode |
| `SECRET_KEY` | In the cloud | Long random string used to sign login sessions |
| `ADMINS` | In the cloud | Secretary accounts, in the format `email\|password\|hostel`. Hostel is `H1`, `H2` or `ALL`. Separate several accounts with `;` |
| `CORS_ORIGIN` | No | Only if the frontend is hosted on a different domain than the API. Also set `API_BASE` in `static/config.js` |
| `SQLITE_PATH` | No | Path of the local SQLite file |

Example `ADMINS` value:

```
h1sec@iitdh.ac.in|a-strong-password|H1;h2sec@iitdh.ac.in|another-strong-password|H2
```

Generate a secret key with:

```bash
python -c "import secrets; print(secrets.token_hex(32))"
```

Never commit real passwords or keys. To change a secretary's password, edit `ADMINS` and redeploy; the stored password is updated on startup.

## API overview

| Method | Endpoint | Who | Purpose |
|---|---|---|---|
| POST | `/api/register` | Anyone | Create a student account |
| POST | `/api/login`, `/api/logout` | Student | Log in and out |
| POST | `/api/admin/login`, `/api/admin/logout` | Secretary | Log in and out |
| GET | `/api/state` | Student / Secretary | Current user's complaints (`?as=admin` for the dashboard) |
| GET | `/api/ping` | Student / Secretary | Cheap change check used for live updates |
| POST | `/api/complaints` | Student | Create a complaint |
| POST | `/api/complaints/<id>/resolve` | Student | Mark resolved or reopen |
| POST | `/api/admin/complaints/<id>` | Secretary | Update status, priority, assignee, remarks |
| GET | `/api/images/<id>` | Owner / Secretary | Fetch a complaint photo |
| GET | `/healthz` | Anyone | Health check for the host |

## Security

- Passwords are hashed (Werkzeug); sessions use HTTP-only cookies, marked secure in the cloud
- Login attempts are limited: 8 failed tries per IP and email lock the account for 10 minutes
- POST requests must be JSON and pass an origin check, which blocks cross-site form posts
- Uploaded photos are validated on the server (JPEG, PNG or WebP, file signature checked, 1.5 MB maximum each, 3 per complaint)
- A student can only see their own complaints and photos; a secretary can only see their own hostel's data
- Ticket numbers are generated with a database row lock, with retries, so two simultaneous submissions never get the same number

## Known limitations

- The login lockout counter is kept in memory, so it is per server process and resets on restart
- Photos are stored in the database, which is simple but grows the database quickly
- Live updates use polling every 8 seconds rather than WebSockets
- No email or WhatsApp notifications yet; the student's WhatsApp number is shown to secretaries for contact

