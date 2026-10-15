# Consignment Note System

Phase 1 sets up the React client and Express API. Phase 2 adds the PostgreSQL schema through Prisma. Consignment save APIs and PDF generation are not implemented yet.

The workspace did not include a transport receipt PDF or screenshot. The sample company name, address, and logo are placeholders. Replace the logo at `client/src/assets/company-logo.svg` and the profile in `client/src/config/company.js`.

The client is bundled with Webpack. Vite is not used.

## Requirements

- Node.js 20 or newer (developed on Node.js 24)
- npm
- A modern browser

PostgreSQL 18 is used for the database. Puppeteer is not required yet.

## Project layout

```text
client/     React application
server/     Express API
```

## Setup

From the project root:

```bash
npm run install:all
```

Or install each app yourself:

```bash
npm install --prefix server
npm install --prefix client
```

Copy the environment examples. The apps also start with the documented defaults if you skip this step.

```bash
copy server\.env.example server\.env
copy client\.env.example client\.env
```

On macOS or Linux, use `cp` instead of `copy`.

### Environment variables

Server (`server/.env`):

| Variable | Purpose |
| --- | --- |
| `PORT` | API port. Default `5000`. |
| `NODE_ENV` | `development` or `production`. |
| `CLIENT_ORIGIN` | Browser origin allowed to call the API directly. Separate multiple origins with commas. |
| `DATABASE_URL` | PostgreSQL connection string used by Prisma. Keep the password in `server/.env` only. |

Client (`client/.env`, read by the dev server and production build):

| Variable | Purpose |
| --- | --- |
| `API_BASE_URL` | API origin for the browser. Leave empty so requests stay relative (`/api/...`). Set this only when the built site is hosted on a different origin from the API. |
| `CLIENT_PORT` | Webpack dev server port. Default `3000`. |
| `API_PROXY_TARGET` | Where the dev server forwards `/api`. Default `http://127.0.0.1:5000`. |

Do not commit `.env` files.

## Run locally

Use two terminals.

Terminal 1, API:

```bash
npm run dev:server
```

Terminal 2, React app:

```bash
npm run dev:client
```

Open `http://127.0.0.1:3000`.

Production API start, after `server/.env` is configured:

```bash
npm start --prefix server
```

Production client build:

```bash
npm run build:client
```

The static files are written to `client/dist`. Serve that folder behind the same origin as the API, or set `API_BASE_URL` at build time to the public API origin. The application code does not hardcode `localhost`.

## Verify the health check

With the API running:

```bash
curl http://127.0.0.1:5000/api/health
```

Expected JSON:

```json
{
  "status": "ok",
  "service": "consignment-note-api",
  "timestamp": "..."
}
```

The dashboard also requests `/api/health` through the dev server proxy and shows whether the API responded.

### Lookup lists

These read-only lists feed the consignment form dropdowns:

```text
GET /api/lookups/freight-types
GET /api/lookups/package-units
GET /api/lookups/parties
GET /api/lookups/items
GET /api/lookups/locations
GET /api/lookups/material-centres
GET /api/lookups/series
GET /api/lookups/mains
GET /api/lookups/types
GET /api/lookups/sale-types
```

Freight types and package units come from application configuration. Parties, goods, locations, and material centres return an empty `not-connected` list until PostgreSQL storage exists. Series, Main, Type, and Sale Type return an empty `unconfigured` list until their allowed values are supplied. None of these responses are saved consignment records.

API tests:

```bash
npm run test:server
```

## What you can try in the browser

- Dashboard shows a zero consignment count, an API status card, and a create action.
- New consignment shows every planned form section, validates required fields, adds and removes goods rows, and totals each goods column separately.
- Reset clears the form. Cancel returns to the dashboard.
- Check form confirms a valid entry and states that nothing was stored.
- History search updates the empty-results message. It does not call a consignment API yet.
- Detail and PDF routes explain that those records are not available yet.

## Database

PostgreSQL must be running locally. This project was checked against the Windows service `postgresql-x64-18`. `psql` is installed at `C:\Program Files\PostgreSQL\18\bin\psql.exe` and is not on the default PATH. Connections use `scram-sha-256`, so a password is required.

Create the application database once. This does not drop or change any existing database:

```text
"C:\Program Files\PostgreSQL\18\bin\psql.exe" -h 127.0.0.1 -U postgres -d postgres -c "CREATE DATABASE consignment_note;"
```

If that database already exists, PostgreSQL reports that and leaves it unchanged. Then set `DATABASE_URL` in `server/.env` using the example in `server/.env.example`.

From the `server` directory, after `DATABASE_URL` points at `consignment_note`:

```bash
npm run prisma:validate
npm run prisma:generate
npm run prisma:migrate
npm run prisma:seed
```

`npm run prisma:migrate` runs `prisma migrate dev` and applies `prisma/migrations`. Do not use `prisma migrate reset`. The seed writes only the placeholder company already shown in `client/src/config/company.js`.

Check the connection without changing data:

```bash
npm run prisma:check
```

That runs `SELECT 1` through Prisma and prints whether the connection succeeded. It does not print `DATABASE_URL`.

## Later phases

Consignment save APIs and Puppeteer PDF rendering are not part of this phase.

When PDF generation is added, the server host will need Chromium dependencies for Puppeteer. On Linux that usually means the shared libraries listed by the Puppeteer install docs for the chosen Puppeteer version, or a container image that already includes them. Windows and macOS can use the browser Puppeteer downloads during `npm install`.
