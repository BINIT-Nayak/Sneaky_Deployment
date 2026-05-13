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

## Public Production Deploy

Use this when you want anyone on the internet to access the app.

### 1. Get a server

Rent a small Ubuntu VPS from any provider. A starter size like 1-2 GB RAM is enough for a small demo, but 2 GB RAM is more comfortable for Docker builds.

Install Docker Engine and the Docker Compose plugin on the server.

### 2. Point your domain to the server

In your domain DNS settings, create an `A` record:

```text
Type: A
Name: @ or sneaky
Value: your_server_public_ip
```

Open inbound firewall ports:

```text
80/tcp
443/tcp
```

### 3. Clone the three repos on the server

Keep this sibling folder layout:

```text
apps/
  Sneaky/
  Sneaky_Backend/
  Sneaky_Deployment/
```

### 4. Create production secrets

From `Sneaky_Deployment`:

```sh
cp .env.example .env
openssl rand -base64 32
openssl rand -base64 24
```

Edit `.env`:

```properties
JWT_SECRET=paste_the_32_byte_secret_here
DOMAIN=sneaky.yourdomain.com
POSTGRES_PASSWORD=paste_the_database_password_here
```

### 5. Start the public app

```sh
docker compose -f docker-compose.prod.yml up --build -d
```

Open:

```text
https://sneaky.yourdomain.com
```

Caddy receives public traffic on ports `80` and `443`, gets/renews HTTPS certificates, and proxies traffic to the frontend container. The frontend Nginx container proxies `/api` to the backend container inside Docker.

Production logs:

```sh
docker compose -f docker-compose.prod.yml logs -f caddy
docker compose -f docker-compose.prod.yml logs -f backend
```

Stop production:

```sh
docker compose -f docker-compose.prod.yml down
```

## Production Notes

`docker-compose.prod.yml` does not publish the backend, PostgreSQL, or Redis directly to the internet. Only Caddy exposes ports `80` and `443`.

Keep `.env` private. Commit `.env.example`, but never commit real `JWT_SECRET` or `POSTGRES_PASSWORD` values.

If you put the frontend and backend behind the same domain and keep Nginx proxying `/api`, CORS and refresh cookies stay much simpler.
