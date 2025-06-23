# High Level Design - Chai ThePeople Client

## Overview
Chai ThePeople is a citizen engagement platform where visitors of local tea stalls answer short surveys in exchange for a free cup of chai. The application connects regular users, shop owners and administrators through a set of role‑based dashboards. Data is stored in Firebase to provide anonymous insights for policy makers while supporting small businesses.

## Architecture
The project is a single page web application built with static HTML, CSS and vanilla JavaScript. User data and survey responses are persisted in Firebase using Firestore and Authentication services. Chart.js is utilised on the admin page for simple visualisations.

```
Browser  <->  Firebase Authentication
        <->  Firestore (users, stalls, questions, user_responses)
```

Pages reference an `auth.js` module (not present in this repo) to initialise Firebase and handle login state. Feature‑specific modules such as `dashboard.js`, `shop.js` and `admin.js` are loaded on their respective dashboards.

## Major Components
- **Home page (`index.html`)**: landing page with simple counters animated via `counters.js`.
- **Authentication pages (`login.html`, `register.html`)**: forms that use Firebase to create or log in users. The register page includes a link to `auth.js` for auth logic【F:register.html†L72-L74】.
- **User dashboard (`user_dashboard.html`)**: displays survey questions filtered for the scanned stall. It loads `auth.js` and `dashboard.js` modules【F:user_dashboard.html†L48-L51】.
- **Shop dashboard (`shop_dashboard.html`)**: allows shop owners to register stalls and view basic statistics. Scripts `auth.js` and `shop.js` are included at the bottom of the page【F:shop_dashboard.html†L100-L105】.
- **Admin dashboard (`admin_dashboard.html`)**: administrators manage questions and users and view response analytics. It loads Chart.js and the modules `auth.js` and `admin.js`【F:admin_dashboard.html†L186-L196】.
- **JavaScript modules**:
  - `admin.js` – handles question management, user role updates and high‑level statistics.
  - `dashboard.js` – fetches questions and submits responses for the user dashboard.
  - `shop.js` – manages stall registration and basic shop statistics.
  - `counters.js` – animates numbers on the landing page.

## Data Model
Firestore holds several collections used by the application:
- `users` – each document represents a user with fields like name, email and role.
- `stalls` – registered stalls with a `stallId` and metadata.
- `questions` – survey questions and any target stall information.
- `user_responses` – answers submitted by users along with the question id and stall context.

These collections are referenced in the JavaScript modules, for example in `admin.js` when populating stall choices or counting responses【F:admin.js†L96-L105】【F:dashboard.js†L223-L241】.

## Technology Stack
As noted in the README, the app relies on a simple stack of HTML/CSS/JavaScript with Firebase providing the backend and Chart.js for charts【F:readme.md†L89-L96】.

## Execution Flow
1. A user visits a stall and scans its QR code leading to `user_dashboard.html?stall=[STALL_ID]`.
2. `auth.js` validates the session or prompts for login/registration.
3. On successful authentication, `dashboard.js` pulls questions from Firestore. Questions are filtered by stall and exclude ones the user has already answered.
4. Submitted answers create documents in the `user_responses` collection.
5. Shop owners and admins see aggregated stats of these responses via their dashboards.

## Deployment Notes
Because ES modules are used, the application must be served over a local web server rather than opened directly from the file system. The README suggests common options such as `python -m http.server` or the VS Code Live Server extension【F:readme.md†L99-L112】.

## Future Improvements
The README outlines potential enhancements such as gamification, advanced analytics, offline support and additional Firebase functions. These can be added incrementally as the system matures【F:readme.md†L240-L256】.

