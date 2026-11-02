# RW PC Trading Web System

RW PC Trading is a browser-based storefront and reservation system for a computer parts and services shop in Valenzuela City. Customers can browse products, check PC-part compatibility, maintain a cart, submit pickup reservations, upload payment proof, save builds, and leave reviews. Staff and administrators can manage catalog data, inventory, reservations, payment verification, services, feedback, reviews, users, and business settings.

The project is implemented as a no-build frontend: static HTML pages and CSS load native JavaScript modules directly in the browser, while Supabase provides authentication, PostgreSQL data storage, row-level security, storage buckets, and server-side inventory functions.

## Contents

- [Core features](#core-features)
- [How it works](#how-it-works)
- [Technology stack](#technology-stack)
- [Project structure](#project-structure)
- [Getting started](#getting-started)
- [Using the application](#using-the-application)
- [Data model and security](#data-model-and-security)
- [Testing and validation](#testing-and-validation)
- [Current status and limitations](#current-status-and-limitations)

## Core features

### Customer experience

- Product catalog with search, category filtering, product details, pricing, warranty information, images, and stock status.
- Service catalog for repair, installation, maintenance, upgrade, and consultation offerings.
- Email/password registration, login, logout, and password reset through Supabase Auth.
- Persistent per-user cart with quantity and stock validation.
- Pickup reservations with a deposit calculated as 20% of the cart total.
- Payment-proof upload and reservation status tracking.
- PC Builder for selecting CPU, motherboard, RAM, GPU, PSU, storage, case, and CPU cooler parts.
- Compatibility checks for CPU sockets, RAM type, case form factor, cooler sockets, GPU clearance, PSU wattage, stock, price, and estimated power draw.
- Saved PC builds on the customer profile.
- Customer reviews with optional image proof and public display after approval.
- Public feedback submission and shop contact information.

### Staff and administrator experience

- Dashboard metrics for products, categories, services, customers, stock, reservations, and pending payment proofs.
- Product and category management, including product archiving and restoration.
- Product and service image uploads through Supabase Storage.
- Inventory adjustments with an audit log.
- Reservation status management and manual payment-proof approval or rejection.
- Atomic reservation approval that checks stock and deducts inventory once.
- Customer profile and staff-role management.
- Review moderation and feedback management.
- Business settings management.

## How it works

1. A visitor opens a static page such as `index.html`, `products.html`, or `pc-builder.html`.
2. The page imports its corresponding module from `js/pages/`.
3. Shared API modules in `js/api/` query Supabase using the browser client in `js/supabaseClient.js`.
4. Supabase Auth maintains the current session. A database trigger creates a `profiles` row for each new account.
5. Row-level security limits customer data to its owner and restricts management actions to staff or administrators.
6. A customer submits a reservation from the cart. The application stores reservation items and calculates the 20% deposit.
7. Staff reviews the uploaded payment proof. Approval calls the `admin_approve_reservation` database function, which verifies stock, deducts inventory, writes inventory logs, and marks the reservation approved.

The application is pickup-oriented. It does not process payments through a payment gateway; payment proof is uploaded and verified manually.

## Technology stack

| Area | Technology |
| --- | --- |
| Frontend | HTML5, CSS3, native ES modules |
| Language | JavaScript |
| Backend platform | Supabase |
| Database | PostgreSQL |
| Authentication | Supabase Auth with email/password |
| File storage | Supabase Storage |
| Client library | `@supabase/supabase-js` v2 loaded from `https://esm.sh` |
| Local development | Any static HTTP server, such as Python's `http.server` |

There is currently no `package.json`, bundler, framework, or build step in the repository.

## Project structure

```text
.
├── index.html                 # Home page and shop contact/feedback form
├── products.html              # Product catalog
├── services.html              # Services catalog
├── pc-builder.html            # Compatibility-aware PC builder
├── cart.html                  # Cart and reservation creation
├── reservations.html          # Customer reservations and payment proofs
├── reviews.html               # Public reviews and customer review form
├── profile.html               # Profile, saved builds, and reservation history
├── login.html
├── register.html
├── admin.html                 # Staff/admin management interface
├── css/
│   └── styles.css
├── js/
│   ├── api/                   # Supabase data-access functions
│   ├── components/            # Shared layout, cards, and inquiry UI
│   ├── pages/                 # Page-specific controllers
│   ├── utils/                 # DOM, formatting, and notifications
│   ├── config.js              # Shop information, roles, statuses, and builder types
│   ├── env.example.js         # Configuration template
│   ├── env.js                 # Local Supabase configuration
│   └── supabaseClient.js      # Supabase client initialization
├── sql/
│   ├── schema.sql             # Tables, functions, policies, storage, and starter records
│   └── sample-data.sql        # Optional richer catalog, services, images, and reviews
└── assets/
    └── images/logo.jpg
```

## Getting started

### Prerequisites

- A Supabase project.
- A modern browser with native JavaScript module support.
- Python 3 or another static HTTP server.

### 1. Configure Supabase

1. Create a Supabase project.
2. In the Supabase SQL Editor, run [`sql/schema.sql`](sql/schema.sql).
3. In Supabase Authentication settings, enable email/password signups.
4. Copy [`js/env.example.js`](js/env.example.js) to `js/env.js`.
5. Replace the placeholder values in `js/env.js` with the Supabase project URL and publishable/anon key:

   ```js
   export const ENV = {
     SUPABASE_URL: "https://YOUR-PROJECT.supabase.co",
     SUPABASE_ANON_KEY: "YOUR-ANON-KEY"
   };
   ```

   Do not commit private keys or service-role credentials. The frontend should use only the public client key.

### 2. Create an administrator

1. Start the application and register the first account through [`register.html`](register.html).
2. Promote that account in the Supabase SQL Editor:

   ```sql
   update public.profiles
   set role = 'admin'
   where email = 'your-email@example.com';
   ```

   Roles defined by the schema are `customer`, `staff`, and `admin`.

### 3. Add optional sample data

Run [`sql/sample-data.sql`](sql/sample-data.sql) after the schema if you want a larger demonstration catalog, services list, product images, and approved reviews. Reviews are linked to existing profiles, so create at least one account before running it if review seed data is required.

### 4. Run the application

From the repository root, start a local HTTP server:

```bash
python -m http.server 5173
```

Open <http://localhost:5173> in a browser. Do not open the HTML files directly from the filesystem because ES modules and Supabase requests require HTTP.

## Using the application

### Customers

1. Browse products or services from the home page.
2. Register or log in to use the cart, reservations, profile, and saved-build features.
3. Use **PC Builder** to select compatible parts, save a build, or add a compatible selection to the cart.
4. Create a pickup reservation from the cart.
5. Upload payment proof from the reservations page. Supported payment method choices in the UI include GCash, Maya, bank transfer, credit card, and e-wallet.
6. Monitor reservation status until staff marks it ready for pickup or claimed.

### Staff and administrators

Users with `staff` or `admin` roles can access `admin.html`. The interface exposes catalog, inventory, service, reservation, payment, customer, review, feedback, user, and settings sections. Category and user-role administration is restricted more narrowly to administrators by the database policies.

## Data model and security

The schema creates these primary tables:

- `profiles`, `categories`, `products`, and `product_images` for identity and catalog data.
- `services` for repair and installation offerings.
- `cart_items` for per-user carts.
- `reservations`, `reservation_items`, and `payment_proofs` for pickup reservations and manual payment verification.
- `inventory_logs` for stock changes and audit history.
- `reviews` and `feedback` for customer communication.
- `saved_builds` for persisted PC configurations.
- `site_settings` for staff-managed business settings.

Supabase row-level security is enabled across the application tables. The schema also defines the `admin_adjust_stock` and `admin_approve_reservation` functions and storage buckets for product images, service images, review proofs, payment proofs, and profile images.

## Testing and validation

No automated test suite or test runner is included in the repository. Manual validation can be performed by:

1. Running the local HTTP server.
2. Registering a customer account and checking login/profile behavior.
3. Loading products and services from Supabase.
4. Testing cart, reservation, payment-proof, and reservation-status workflows.
5. Promoting a test profile to `staff` or `admin` and checking the management pages.
6. Testing PC Builder warnings with mismatched sockets, RAM, case clearance, and PSU wattage.

## Current status and limitations

This is an implemented browser-based storefront prototype/application that requires a configured Supabase project and database before live data features work. The repository currently has these operational limitations:

- There is no payment gateway integration; payment verification is manual.
- The reservation workflow is pickup-only.
- There is no automated test suite, dependency lockfile, or production build pipeline.
- Product and service image seed data uses externally hosted image URLs; replace or manage these through the admin interface for production use.
- Supabase configuration is supplied through a local JavaScript module rather than a server-side environment-variable process.

## License and credits

No license file or project-specific author/credit information is included in the repository. Add those details if this project is intended for public distribution or portfolio publication.
