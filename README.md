# FastAPI Blog

A server-rendered blog application and JSON API built with FastAPI, SQLAlchemy 2, Jinja2, and asynchronous database access. Users can register, authenticate with JWT bearer tokens, create and manage posts, update their account, upload profile pictures, and reset forgotten passwords.

## Features

- Server-rendered pages for the home feed, posts, user posts, authentication, and account management
- JSON API with automatic OpenAPI documentation
- JWT authentication with Argon2 password hashing
- Post CRUD with ownership checks and pagination
- User registration, profile updates, password changes, and password resets
- Profile-picture processing with Pillow and storage in an S3-compatible bucket
- Async SQLAlchemy sessions with Alembic migrations
- SMTP email delivery for password-reset messages

## Requirements

- Python 3.12+
- [uv](https://docs.astral.sh/uv/) (recommended)
- A database supported by SQLAlchemy, such as SQLite or PostgreSQL
- An S3-compatible bucket for profile pictures
- An SMTP server if password-reset emails are required

## Getting started

### 1. Install dependencies

```bash
uv sync
```

### 2. Configure environment variables

Create a `.env` file in the project root. The file is ignored by Git, so do not commit real credentials.

```dotenv
SECRET_KEY=replace-with-a-long-random-secret
DATABASE_URL=sqlite+aiosqlite:///./blog.db

S3_BUCKET_NAME=your-bucket-name
S3_REGION=us-east-2
S3_ACCESS_KEY_ID=your-access-key        # optional when using an IAM role or local AWS config
S3_SECRET_ACCESS_KEY=your-secret-key     # optional when using an IAM role or local AWS config
# S3_ENDPOINT_URL=http://localhost:9000  # optional, useful for MinIO or another S3-compatible service

MAIL_SERVER=smtp.example.com
MAIL_PORT=587
MAIL_USERNAME=your-smtp-username
MAIL_PASSWORD=your-smtp-password
MAIL_FROM=noreply@example.com
MAIL_USE_TLS=true

FRONTEND_URL=http://localhost:8000
```

`DATABASE_URL` must use an async SQLAlchemy driver. For PostgreSQL, for example:

```dotenv
DATABASE_URL=postgresql+psycopg://username:password@localhost:5432/blog
```

The application also supports optional settings such as `ALGORITHM`, `ACCESS_TOKEN_EXPIRE_MINUTES`, `MAX_UPLOAD_SIZE_BYTES`, `POSTS_PER_PAGE`, and `RESET_TOKEN_EXPIRE_MINUTES`. Their defaults are defined in `config.py`.

### 3. Create or update the database

```bash
uv run alembic upgrade head
```

### 4. Start the development server

```bash
uv run fastapi dev main.py
```

Open [http://localhost:8000](http://localhost:8000) in a browser. Interactive API documentation is available at [http://localhost:8000/docs](http://localhost:8000/docs), with the raw OpenAPI schema at `/openapi.json`.

## API overview

All API routes are prefixed with `/api`.

| Method | Endpoint | Auth | Description |
| --- | --- | --- | --- |
| `POST` | `/api/users` | No | Register a user |
| `POST` | `/api/users/token` | No | Log in and receive a JWT; submit OAuth2 form fields `username` (email) and `password` |
| `GET` | `/api/users/me` | Bearer token | Get the current user |
| `GET` | `/api/users/{user_id}` | No | Get a public user profile |
| `PATCH` | `/api/users/{user_id}` | Owner | Update username or email |
| `DELETE` | `/api/users/{user_id}` | Owner | Delete the current user |
| `GET` | `/api/users/{user_id}/posts` | No | List a user’s posts |
| `PATCH` | `/api/users/{user_id}/picture` | Owner | Upload a profile picture as multipart form data using the `file` field |
| `DELETE` | `/api/users/{user_id}/picture` | Owner | Remove a profile picture |
| `POST` | `/api/users/forgot-password` | No | Request password-reset instructions |
| `POST` | `/api/users/reset-password` | No | Set a new password with a reset token |
| `PATCH` | `/api/users/me/password` | Bearer token | Change the current password |
| `GET` | `/api/posts` | No | List posts; supports `skip` and `limit` |
| `POST` | `/api/posts` | Bearer token | Create a post |
| `GET` | `/api/posts/{post_id}` | No | Get a post |
| `PUT` | `/api/posts/{post_id}` | Post owner | Replace a post |
| `PATCH` | `/api/posts/{post_id}` | Post owner | Partially update a post |
| `DELETE` | `/api/posts/{post_id}` | Post owner | Delete a post |

Pagination endpoints return `posts`, `total`, `skip`, `limit`, and `has_more`. Post titles are limited to 100 characters; passwords must contain at least 8 characters.

### Example API usage

Register and log in:

```bash
curl -X POST http://localhost:8000/api/users \
  -H "Content-Type: application/json" \
  -d '{"username":"jane","email":"jane@example.com","password":"correct-horse-battery"}'

curl -X POST http://localhost:8000/api/users/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "username=jane@example.com&password=correct-horse-battery"
```

Use the returned access token to create a post:

```bash
curl -X POST http://localhost:8000/api/posts \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"My first post","content":"Hello from FastAPI!"}'
```

## Web pages

- `/` or `/posts` — home feed
- `/posts/{post_id}` — post page
- `/users/{user_id}/posts` — posts by a user
- `/login` — login page
- `/register` — registration page
- `/account` — account page
- `/forgot-password` — password-reset request page
- `/reset-password` — password-reset form

## Database migrations

Create a migration after changing the SQLAlchemy models:

```bash
uv run alembic revision --autogenerate -m "describe the change"
uv run alembic upgrade head
```

To inspect the current migration revision:

```bash
uv run alembic current
```

## Project layout

```text
main.py             FastAPI app and server-rendered page routes
routers/            User and post API routers
models.py           SQLAlchemy models
schemas.py          Pydantic request and response schemas
database.py         Async engine, sessions, and declarative base
auth.py             Password hashing and JWT authentication
config.py           Environment-backed application settings
templates/          Jinja2 HTML templates
static/             CSS, JavaScript, icons, and default assets
alembic/            Database migration configuration and revisions
```

## Production notes

- Set a strong, unique `SECRET_KEY` and keep all credentials outside version control.
- Use a managed PostgreSQL database and an S3 bucket or compatible object store for production deployments.
- Configure a real SMTP provider before enabling password resets.
- Review the S3 bucket’s public-read policy or replace the generated image URL behavior if profile pictures should not be public.
- Run migrations as part of deployment before starting the application.
