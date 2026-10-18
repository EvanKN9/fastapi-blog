# FastAPI Blog

A full-stack blog application built with **FastAPI**, **SQLAlchemy**, and **PostgreSQL**. The application supports user authentication, blog post management, profile customization, and password recovery through both server-rendered pages and a REST API.

Containerized with **Docker** and deployed to **Google Cloud Run**, using **Neon PostgreSQL** for database hosting and **Amazon S3** for profile picture storage.

This project was developed by following a FastAPI tutorial, with the goal of gaining hands-on experience in backend development, database management, authentication, testing, containerization, and cloud deployment.

## Live Demo

**Application:** [View Live Blog](https://fastapi-service-1017307243746.us-west1.run.app)

**API Documentation:** [Swagger UI](https://fastapi-service-1017307243746.us-west1.run.app/docs)

## Features

### Authentication and User Management
- User registration and login
- JWT-based authentication
- Secure password hashing using Argon2
- Profile updates and password changes
- Password reset functionality via email
- Profile picture uploads using Amazon S3

### Blog Management
- Create, read, update, and delete blog posts
- Post ownership and authorization checks
- Paginated post listings
- User-specific blog posts
- Server-rendered pages using Jinja2
- REST API with automatically generated OpenAPI documentation

### Backend and Infrastructure
- Asynchronous database operations using SQLAlchemy
- PostgreSQL database hosted on Neon
- Database schema migrations using Alembic
- Environment-based configuration using Pydantic Settings
- Automated API testing with pytest
- Containerization using Docker
- Cloud deployment using Google Cloud Run

## Tech Stack

| Category | Technologies |
|---|---|
| Backend | Python 3.12, FastAPI |
| Frontend | HTML, Jinja2 |
| Database | PostgreSQL, Neon |
| ORM | SQLAlchemy 2.0 |
| Migrations | Alembic |
| Authentication | JWT, Argon2 |
| Object Storage | Amazon S3, Boto3 |
| Email | aiosmtplib |
| Testing | pytest, Moto |
| Dependency Management | uv |
| Containerization | Docker |
| Cloud Hosting | Google Cloud Run |
| Image Registry | Google Artifact Registry |
| Image Builds | Google Cloud Build |

## Deployment Architecture

The application uses a containerized deployment architecture:

```text
             Client / Browser
                    |
                    | HTTPS
                    v
            Google Cloud Run
                    |
             FastAPI Application
                    |
          +---------+---------+
          |         |         |
          v         v         v
        Neon      AWS S3    SMTP
      PostgreSQL  Storage   Service
```

- **Google Cloud Run** hosts the containerized FastAPI application.
- **Neon** provides a managed PostgreSQL database.
- **Amazon S3** stores uploaded profile pictures.
- **Google Artifact Registry** stores Docker images.
- **Google Cloud Build** builds the container image for deployment.

## Getting Started

### Prerequisites

- Python 3.12+
- [uv](https://docs.astral.sh/uv/)
- PostgreSQL or SQLite for local development
- AWS S3 credentials for profile picture uploads
- SMTP configuration for password reset functionality

### Installation

**1. Clone the repository**

```bash
git clone https://github.com/YOUR_USERNAME/fastapi-blog.git
cd fastapi-blog
```

**2. Install dependencies**

```bash
uv sync
```

This creates a virtual environment and installs the project dependencies from `uv.lock`.

**3. Configure environment variables**

Create a `.env` file in the project root and configure the settings required by the application, including:

- Database connection URL
- JWT secret key
- AWS S3 credentials and bucket configuration
- SMTP server credentials

Refer to the application's configuration module for the exact variable names.

**4. Apply database migrations**

```bash
uv run alembic upgrade head
```

**5. Start the application**

```bash
uv run fastapi dev
```

Open the local application URL shown in your terminal, typically:

```text
http://127.0.0.1:8000
```

Interactive API documentation is available at `/docs`.

## Running with Docker

Build the Docker image:

```bash
docker build -t fastapi-blog .
```

Run the application with environment variables from your local `.env` file:

```bash
docker run --env-file .env -p 8080:8080 fastapi-blog
```

The application should be accessible at:

```text
http://localhost:8080
```

The Dockerfile uses a multi-stage build to separate dependency installation from the final runtime image and runs the application as a non-root user.

## Testing

The project includes automated API tests using **pytest**, covering functionality such as:

- Retrieving blog posts
- Creating posts
- Updating and deleting posts
- Authentication requirements
- Post ownership and authorization
- Error handling

Run the test suite with:

```bash
uv run pytest tests/ -v
```

## What I Learned

Through this project, I gained practical experience with:

- Designing REST APIs using FastAPI and Pydantic
- Implementing authentication and authorization
- Working with relational databases through SQLAlchemy
- Managing database migrations with Alembic
- Writing automated API tests
- Integrating external services such as Amazon S3
- Building multi-stage Docker images
- Deploying containerized applications to Google Cloud Run
- Configuring a managed PostgreSQL database using Neon
- Managing environment variables and production configuration

## Acknowledgments

This application was developed by following a FastAPI tutorial. The project served as a hands-on learning exercise for exploring backend engineering and cloud deployment.

**Tutorial:** [https://www.youtube.com/watch?v=iukOehU5aF4]

## License

This project is intended for educational and portfolio purposes. See the repository's license file, if provided, for usage terms.
