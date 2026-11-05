# Brain TDS Logistics API

This folder contains the standalone Node.js/Express backend. Its database is the PostgreSQL database included with Supabase, accessed from Node through the `pg` driver. The frontend is a separate project at `../brain-tds-logistics`.

## Requirements

- Node.js 20 or newer
- A Supabase project

## Setup

1. Run `npm install` in this folder.
2. In your Supabase project's SQL Editor, run `supabase/schema.sql`.
3. In Supabase, open **Project Settings → Database** and copy a PostgreSQL connection string. Use the session pooler connection if your network does not support IPv6, and URL-encode any special characters in the database password.
4. Copy `.env.example` to `.env`; set `SUPABASE_DB_URL`, `SUPABASE_URL`, and `SUPABASE_ANON_KEY`. The anon/publishable key is used only to call Supabase Auth; the database password stays on the server.
5. In Supabase Auth URL configuration, allow the frontend URL and password reset redirect URL. Set `PASSWORD_RESET_REDIRECT_URL` to the frontend account page; the recovery link returns to that page and displays the new-password form.
6. Start the API with `npm run dev`. By default it listens on `http://localhost:4000`.
7. In another terminal, start the frontend from `../brain-tds-logistics` using its `npm run dev` command.

The default `CORS_ORIGINS` allows the local frontend on ports 3000 and 3123. Set it to the exact origin(s) where your frontend is hosted when deploying. The frontend's `api-base-url` meta tag can be set to the deployed API URL; when blank, it uses port 4000 for localhost and the same origin otherwise.

`npm start` starts the API for production, and `npm test` runs the API tests. Without Supabase credentials, health checks still work and database/auth endpoints return `503 Service Unavailable`.

## API

- `GET /api/health` — reports API status and whether the PostgreSQL connection is configured.
- `POST /api/auth/signup`, `/api/auth/login`, `/api/auth/refresh`, `/api/auth/recovery`, `PATCH /api/auth/password`, `GET /api/auth/me` — customer account registration, authentication, and password recovery.
- `GET /api/shipments/:trackingNumber` — returns the matching shipment and up to 10 latest tracking events.
- `POST /api/inquiries` — saves a shipment request. The JSON body requires `fullName`, `email`, `phone`, `origin`, and `cargoDescription`.
- `GET /api/customer/dashboard`, `POST /api/customer/bookings` — account-specific shipment and booking management.
- `GET /api/staff/dashboard`, `PATCH /api/staff/inquiries/:id`, `POST /api/staff/shipments`, `POST /api/staff/shipments/:id/events` — staff shipment and inquiry management.
- `GET /api/admin/users`, `PATCH /api/admin/users/:id/role` — administrator account-role management.

After creating your account, promote the first staff administrator in Supabase SQL Editor with `update public.profiles set role = 'admin' where email = 'your-admin-email@example.com';`. New accounts are always customers; never accept an account role from signup input. Supabase Auth verifies bearer tokens, and the API authorizes customer/staff/admin operations against `public.profiles`. Use a least-privilege PostgreSQL role for the backend connection; never put the database connection string in frontend code.
