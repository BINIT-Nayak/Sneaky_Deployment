# Sneaky Docker Setup

This runs the full stack:

- React/Vite frontend on `http://localhost:5173`
- Spring Boot backend on `http://localhost:8081`
- PostgreSQL on `localhost:5432`
- Redis on `localhost:6379`

## 1. Create your Docker env file

From `/Users/ar-in-m-049/Desktop/personal/Sneaky_Deployment`:

```sh
cp .env.example .env
openssl rand -base64 32
```

Paste the generated value into `JWT_SECRET` in `.env`.

## 2. Build and start everything

```sh
docker compose up --build
```

Open:

```text
http://localhost:5173
```

The frontend container serves the React build with Nginx. Browser requests to `/api/...` are proxied inside Docker to the backend service at `backend:8081`, so `VITE_API_BASE_URL` stays empty.

This deployment repo expects the frontend and backend repos to be sibling folders:

```text
personal/
  Sneaky/
  Sneaky_Backend/
  Sneaky_Deployment/
```

## Useful commands

```sh
docker compose up --build -d
docker compose logs -f backend
docker compose logs -f frontend
docker compose ps
docker compose down
```

To delete the database and Redis volumes too:

```sh
docker compose down -v
```

## Production notes

Before deploying to a real server, change the database password in `docker-compose.yml`, use a strong `JWT_SECRET`, and update:

```yaml
APP_CORS_ALLOWED_ORIGINS: https://your-domain.com
APP_AUTH_REFRESH_COOKIE_SECURE: "true"
APP_AUTH_REFRESH_COOKIE_SAME_SITE: None
```

If you put the frontend and backend behind the same domain and keep Nginx proxying `/api`, CORS and refresh cookies stay much simpler.
