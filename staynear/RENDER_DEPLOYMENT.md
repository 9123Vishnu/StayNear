# StayNear deployment guide (Render + managed MySQL)

## Architecture
- Frontend: Render Static Site from `frontend/`
- Backend: Render Web Service using Docker from `backend/project/`
- Database: managed MySQL provider reachable from the Render backend

Render does not provide a native managed MySQL database service in the same way it does for PostgreSQL. Use a managed MySQL provider and follow its network access and TLS instructions.

## 1. Prepare MySQL
1. Create a MySQL database named `staynear` and a dedicated application user (avoid root in production).
2. Save the host, port, database name, username, and password.
3. Use the provider's required TLS settings in `SPRING_DATASOURCE_URL`, for example `jdbc:mysql://HOST:3306/staynear?sslMode=REQUIRED` only if supported by that provider.
4. Back up any existing database before migration. The application contains a startup migration that alters three columns; test it against a staging database first.

## 2. Push the project to GitHub
Commit the project files, but never commit `.env`, production passwords, or database URLs containing credentials. Use `.env.example` as a template only.

## 3. Deploy backend on Render
1. Create **New > Web Service** and connect the repository.
2. Choose Docker runtime.
3. Set Dockerfile path to `backend/project/Dockerfile` and Docker context/root directory to `backend/project` (Render dashboard labels can differ; ensure the Docker build context contains `pom.xml` and `src/`).
4. Add environment variables:
   - `SPRING_DATASOURCE_URL` = provider JDBC URL
   - `SPRING_DATASOURCE_USERNAME` = app database username
   - `SPRING_DATASOURCE_PASSWORD` = app database password
   - `SPRING_JPA_HIBERNATE_DDL_AUTO` = `update` for initial testing only; use migrations and `validate` for production
   - `APP_CORS_ALLOWED_ORIGINS` = exact frontend URL after frontend is deployed
5. Deploy and copy the public backend URL.

The app uses `${PORT:8080}`, so it respects Render's assigned `PORT`.

## 4. Deploy frontend on Render
1. Create **New > Static Site** using the same repository.
2. Root directory: `frontend`.
3. Build command: `npm ci && npm run build`.
4. Publish directory: `dist`.
5. Add build environment variable `VITE_API_BASE_URL` = `https://YOUR-BACKEND.onrender.com/api`.
6. Deploy. Vite variables are baked in during build, so redeploy after changing this value.

If the site uses React Router, configure a rewrite of `/*` to `/index.html` with status `200` in Render's Redirects/Rewrites settings.

## 5. Set CORS and verify
After obtaining the frontend URL, set backend `APP_CORS_ALLOWED_ORIGINS` to that exact origin, such as `https://YOUR-FRONTEND.onrender.com`, with no trailing slash, then redeploy the backend.

Test signup/login, owner property creation, company tagging, tenant searches, browser refresh, and database persistence. Check Render logs and browser devtools for failures.

## Local Docker
1. Copy `.env.example` to `.env` and change the local password.
2. Run `docker compose up --build` from the repository root.
3. Open `http://localhost:5173`.
4. Stop with `docker compose down`. Avoid `docker compose down -v` unless you intend to delete the database volume.

## Important security limitations before public production
- `SecurityConfig` currently permits all HTTP requests. BCrypt password hashing is present, but the app does not yet authenticate/authorize subsequent API calls or enforce owner-specific access on every property operation. Do not use this as a public production service with real user data until JWT/session authentication and resource-level authorization are implemented and tested.
- Do not use a root MySQL account in production.
- Use a migration tool (Flyway/Liquibase) and set Hibernate DDL mode to `validate` after schema migration is established.
- Add rate limiting, validation, centralized error handling, backups, monitoring, and an appropriate privacy policy before launch.
