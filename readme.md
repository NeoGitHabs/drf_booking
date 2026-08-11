# Hotel & Apartment Booking API

> Production-ready REST API for hotel and apartment reservations —
> real-time availability validation, role-based access, and multilingual
> content for guest and owner workflows.

[![Python](https://img.shields.io/badge/Python-3.11-blue)]()
[![Django](https://img.shields.io/badge/Django-5.2-green)]()
[![DRF](https://img.shields.io/badge/DRF-3.16-red)]()
[![Docker](https://img.shields.io/badge/Docker-Compose-blue)]()
[![JWT](https://img.shields.io/badge/Auth-JWT-yellow)]()
[![License: MIT](https://img.shields.io/badge/License-MIT-green)]()

---

## Problem

Hospitality platforms lose revenue when double-bookings occur or inventory
status drifts under concurrent requests. This API centralizes all booking
logic with atomic validation — availability is enforced at the data layer,
not just the UI.

---

## Endpoints

| Method | URL | Description |
|--------|-----|-------------|
| POST | `/register/` | Register (guest/owner) |
| POST | `/login/` | JWT login |
| POST | `/logout/` | Blacklist token |
| GET | `/` | Browse cities |
| GET | `/hotel/` | List hotels |
| GET | `/hotel/<pk>/` | Hotel apartments |
| POST | `/review/` | Leave a review |
| GET | `/review/<pk>/` | Hotel reviews |
| POST | `/mysite/` | Create booking |
| DELETE | `/mysite/<pk>/cancel/` | Cancel booking |
| GET/POST | `/manage_hotel/` | Owner: manage hotels |
| GET/POST | `/manage_apartment/` | Owner: manage apartments |
| POST | `/become_owner/` | Upgrade to owner role |
| GET | `/favorite/` | Favorites list |

Swagger UI: `http://localhost/en/api/docs/`

---

## Quick Start

```bash
git clone https://github.com/your-username/hotel-booking-api
cd hotel-booking-api
cp .env.example .env   # fill SECRET_KEY and DB credentials
docker-compose up --build
```

```bash
# Optional: create superuser
docker-compose exec web python manage.py createsuperuser
```

API: `http://localhost/en/`
Swagger: `http://localhost/en/api/docs/`

---

## Demo

**Register:**
```bash
curl -X POST http://localhost/en/register/ \
  -H "Content-Type: application/json" \
  -d '{"username": "alice", "email": "alice@example.com",
       "password": "secret123", "user_phone_number": "+12025551234",
       "guest_status": "guest"}'
```
```json
{
  "user": {"username": "alice", "email": "alice@example.com"},
  "access": "<JWT_ACCESS_TOKEN>",
  "refresh": "<JWT_REFRESH_TOKEN>"
}
```

**Create booking:**
```bash
curl -X POST http://localhost/en/mysite/ \
  -H "Authorization: Bearer <access_token>" \
  -H "Content-Type: application/json" \
  -d '{"hotel_reservation": 1, "apartment_reservation": 3,
       "check_in_date": "2025-09-01", "check_out_date": "2025-09-05"}'
```
```json
{
  "id": 7,
  "apartment_reservation": 3,
  "check_in_date": "2025-09-01",
  "check_out_date": "2025-09-05"
}
```

**Cancel booking:**
```bash
curl -X DELETE http://localhost/en/mysite/7/cancel/ \
  -H "Authorization: Bearer <access_token>"
```

---

## Tech Stack

| Category | Tools |
|----------|-------|
| Language | Python 3.11 |
| Framework | Django 5.2, DRF 3.16 |
| Auth | SimpleJWT + blacklist, django-allauth (OAuth) |
| Database | PostgreSQL (prod) / SQLite (dev) |
| Filtering | django-filter, OrderingFilter, SearchFilter |
| i18n | django-modeltranslation (EN / RU / ES) |
| API Docs | drf-spectacular (Swagger UI) |
| Deploy | Docker Compose, Gunicorn, Nginx |
| Media | Pillow, shared Docker volume |
| Config | python-dotenv |

---

## Project Structure
```
drf_booking/
    ├── .gitignore
    ├── readme.md
    └── booking_pro
        ├── Dockerfile
        ├── db.sqlite3
        ├── docker-compose.yml
        ├── manage.py
        ├── requirements.txt
        ├── booking
        │   ├── __init__.py
        │   ├── asgi.py
        │   ├── settings.py
        │   ├── urls.py
        │   └── wsgi.py
        ├── media
        ├── mysite
        │   ├── __init__.py
        │   ├── admin.py
        │   ├── apps.py
        │   ├── filters.py
        │   ├── migrations
        │   ├── models.py
        │   ├── permissions.py
        │   ├── serializers.py
        │   ├── tests.py
        │   ├── translation.py
        │   ├── urls.py
        │   └── views.py
        ├── nginx
        │   ├── Dockerfile
        │   └── nginx.conf
        └── документация
            └── Booking.com.docx
```

---

## Key Decisions

**Double-booking protection**
Validation runs in both `serializer.validate()` AND `model.save()` —
two-layer defense; only one booking succeeds even under concurrent load.

**Apartment status sync**
`Booking.delete()` overridden to reset `apartment.is_free = 'available'`
before deletion — status always reflects real availability.

**Self-review prevention**
Guard added in both `ReviewsSerializer.validate()` and
`perform_create()` — returns 400 with explicit error, never saves
corrupt data.

**Role permissions**
Custom permission classes: `CheckRole`, `CheckUserRoleReviews`,
`IsHotelOwner` — owners manage hotels, guests make bookings.

---
