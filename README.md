# Hairitage Studio

Website, booking system and content admin for **Hairitage Studio**, a hair salon in South Africa.
Live at **[hairitage-studio.co.za](https://hairitage-studio.co.za)**.

Built and run end to end by me: Django application, admin tooling, WhatsApp/email
integrations, and the Google Cloud infrastructure it runs on.

![Python](https://img.shields.io/badge/Python-3.11-3776AB?logo=python&logoColor=white)
![Django](https://img.shields.io/badge/Django-092E20?logo=django&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-15-4169E1?logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?logo=docker&logoColor=white)
![Google Cloud](https://img.shields.io/badge/Cloud_Run-4285F4?logo=googlecloud&logoColor=white)

## What it does

**For clients**
- Marketing site: services, portfolio, pricing, promotions, team, FAQ, blog
- Online appointment booking with confirmation emails
- Automatic **WhatsApp and email reminders** 30 and 5 minutes before an appointment
- Contact form protected by a honeypot, signed form tokens and Cloudflare Turnstile

**For the salon (admin)**
- Every page is editable from the Django admin (Jazzmin theme, TinyMCE rich text):
  services, price lists, portfolio, blog posts, legal pages, site settings
- Inbox for contact messages and replies, appointment list, prospect tracking
- WhatsApp delivery log fed by a Meta webhook (sent / delivered / read / failed)

## Architecture

```
                 ┌──────────── Google Cloud (africa-south1) ────────────┐
 Browser ──HTTPS──► Load balancer ─┬─► Cloud Run: Django + gunicorn ───┼──► Cloud SQL (PostgreSQL)
                 │                 └─► Cloud Storage (static & media)  │
                 │   Secret Manager: DB, SMTP, Turnstile, admin creds  │
                 └──────────────────────────────────────────────────────┘
                           │                         │
                    Gmail SMTP (email)     Meta WhatsApp Cloud API (reminders + webhook)
```

| Area | Choice |
|---|---|
| Backend | Django, gunicorn, PostgreSQL |
| Apps | `work` (services, portfolio, team) · `pricing` · `product` (promotions) · `blog` · `contact` (messages, appointments, reminders) · `legal` · `configuration` (site settings, stats, FAQ) · `log` (WhatsApp status) |
| Frontend | Server-rendered Django templates, Bootstrap |
| Hosting | Cloud Run + Cloud SQL + Cloud Storage behind an HTTPS load balancer; secrets in Secret Manager, never in the repo |
| Alternate hosting | Docker Compose + nginx + Let's Encrypt (`docker-compose.prod.yml`, `deploy/lightsail-setup.sh`) |

## Run it locally

Requires Docker.

```bash
cp .env.db.local.example .env.db.local     # edit values if you like
docker compose up --build
docker compose exec web python manage.py createsuperuser
docker compose exec web python manage.py seed_site      # optional sample content
```

| URL | |
|---|---|
| http://localhost:1337 | Site (through nginx) |
| http://localhost:1337/admin/ | Admin |
| http://localhost:7580 | Adminer (database browser) |

Email, WhatsApp, Turnstile and Google Maps are optional; leave their keys blank and the
site falls back gracefully.

## Deploying to Google Cloud

The scripts in [`deploy/gcp`](deploy/gcp) provision and deploy everything with `gcloud`.
Each has a `.sh` and a Windows `.bat` version.

```bash
cp deploy/gcp/config.example.sh deploy/gcp/config.sh   # set your project id, region, domain
bash deploy/gcp/setup-infra.sh      # APIs, service accounts, Cloud SQL, bucket, secrets
bash deploy/gcp/deploy.sh           # build with Cloud Build and deploy to Cloud Run
bash deploy/gcp/migrate.sh          # run migrations as a Cloud Run job
bash deploy/gcp/setup-alb.sh        # HTTPS load balancer + managed certificate
```

Other scripts cover changing email or Turnstile settings without a rebuild, rotating the
database and admin passwords, seeding content and importing data from a local database.

## Project layout

```
app/
  hairitage/        settings, URLs, WSGI
  work/ pricing/ product/ blog/ contact/ legal/ configuration/ log/ common/
  templates/site/   pages and email templates
  Dockerfile        local / Compose image
  Dockerfile.cloudrun, entrypoint.cloudrun.sh, cloudbuild.yaml
deploy/gcp/         Google Cloud provisioning and deploy scripts
nginx/              nginx configs for the Compose setup
```
