# Hostel Complaints: central cloud system

Student devices -> this website -> Flask API -> ONE Postgres database <- secretary dashboard.
No complaint data is stored in the browser. Everything is saved on the server.

## 1. Try it on your laptop
    python -m venv venv
    venv\Scripts\activate        (Mac/Linux: source venv/bin/activate)
    pip install -r requirements.txt
    python app.py
Students: http://127.0.0.1:5000   Secretary: http://127.0.0.1:5000/admin   (dev login: sec@h1.edu / secretary123)
Local mode uses a SQLite file (hostel.db). It is for testing only.

## 2. Put it in the cloud (Render example)
1. Push this folder to a GitHub repository.
2. Render > New > Blueprint > select the repo (uses render.yaml). It creates the web service and a Postgres database.
3. In the Render dashboard set ADMINS, for example:
   h2sec@iitdh.ac.in|YourStrongPassword|H2
   (format email|password|hostel, separate several with ;  use ALL instead of H2 to see both hostels)
4. Open the URL Render gives you. Students use it as is; the secretary opens /admin.
Any host that runs Python + Postgres works the same way (Railway, Fly.io, Heroku, a VPS):
set DATABASE_URL, SECRET_KEY and ADMINS, then run: gunicorn app:app

## Environment variables
DATABASE_URL  Postgres connection string (setting it switches the app to cloud mode)
SECRET_KEY    long random string (required in cloud mode)
ADMINS        secretary accounts, see above (required in cloud mode)
CORS_ORIGIN   only if the website is hosted on a different domain than the API; also set API_BASE in static/config.js
SQLITE_PATH   optional path for the local SQLite file

## How updates reach the secretary
The dashboard asks the server every 8 seconds whether anything changed and redraws automatically
(it never redraws while you are typing or have a complaint open). New complaints show a "New complaint received" message.
