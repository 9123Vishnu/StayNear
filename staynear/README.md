# StayNear — Rental discovery and multi-company tagging

StayNear is a React/Vite frontend with a Java 21 / Spring Boot REST API and MySQL database. Owners can create listings and tag companies; tenants can search listings by company.

## Run locally with Docker (recommended)
1. Install and start Docker Desktop.
2. Copy `.env.example` to `.env` and replace the sample `MYSQL_ROOT_PASSWORD` with a strong local password.
3. From the repository root (where `docker-compose.yml` lives), run:

   ```bash
   docker compose up --build
   ```

4. Open `http://localhost:5173`. The backend is exposed at `http://localhost:8080/api`.
5. Stop services with `docker compose down`. This preserves the database volume. `docker compose down -v` deletes that volume and its data.

## Run without Docker
- Start MySQL and create a database named `staynear`.
- Set `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, and `SPRING_DATASOURCE_PASSWORD` in your shell/environment.
- From `backend/project`, run `./mvnw spring-boot:run` (Windows: `mvnw.cmd spring-boot:run`).
- From `frontend`, run `npm ci` then `npm run dev`.

## Public deployment
See [RENDER_DEPLOYMENT.md](RENDER_DEPLOYMENT.md) for Render Static Site + Docker Web Service + managed MySQL steps.

## API overview
- `POST /api/auth/signup`
- `POST /api/auth/login`
- `GET, POST /api/companies`
- `POST /api/properties`
- `GET /api/properties/nearby`
- `GET /api/properties/owner/{ownerId}`
- `GET /api/properties/{id}`
- `PUT, DELETE /api/properties/{id}`
- `GET, POST /api/properties/{propertyId}/companies`
- `DELETE /api/properties/{propertyId}/companies/{companyId}`

## Production warning
This is a deployable development/MVP configuration, not yet safe for a public production launch. `SecurityConfig` currently permits all requests, so authentication and owner-level authorization must be implemented and tested before real users or personal data are exposed. Use a dedicated MySQL user, TLS, managed secrets, backups, and schema migrations in production.
