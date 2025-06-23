# Chai ThePeople Client-Side Low Level Design

This document describes the detailed design of the front end application contained in this repository.  The project provides static HTML pages with JavaScript that interacts with Firebase Authentication and Firestore to implement survey functionality for users, shop owners and administrators.

## 1. Overall Architecture

```
[Browser] --(HTML/CSS/JS)--> [Firebase Auth & Firestore]
```

All pages load JavaScript modules which import Firebase libraries directly from the CDN.  A common `auth.js` module (not committed in the repository) initialises Firebase, manages authentication state and exposes two named exports:

- `auth` – instance of `getAuth()`
- `db` – instance of `getFirestore()`

Each dashboard page imports `auth.js` plus a page specific module (`dashboard.js`, `shop.js` or `admin.js`).  The modules use Firebase Firestore collections:

- `users` – profile and role data
- `stalls` – tea stall records
- `questions` – survey questions
- `user_responses` – survey answers

## 2. Page Level Components

### 2.1 index.html
* Landing page introducing the project.
* Contains counters animated by `counters.js`.
* Navigation links to login/register and each dashboard.  Links are shown/hidden by `auth.js` based on user role.

### 2.2 login.html / register.html
* Simple forms that post user data to functions inside `auth.js` for sign‑in or sign‑up.
* After authentication the user is redirected depending on their role.

### 2.3 user_dashboard.html + dashboard.js
* Shows available survey questions.
* URL parameter `?stall=[STALL_ID]` sets the context for stall‑specific questions.
* `dashboard.js` logic:
  - `getQueryParam(param)` – returns value of a query parameter.
  - `getAnsweredQuestionIds(userId)` – fetches from `user_responses` collection and builds a set of question IDs the user has already answered.
  - `renderQuestion(questionId, questionData)` – creates DOM elements for a question and its options.
  - `displayQuestions()` – queries `questions` collection for global and stall specific questions, filters out answered ones and renders them.
  - `handleAnswerSubmit(event)` – submits a selected option to `user_responses` and updates the UI.
  - Auth state listener triggers page initialisation only once per session.

### 2.4 shop_dashboard.html + shop.js
* Accessible to users with role `shop`.
* Key functions inside `shop.js`:
  - `showStatus(element, message, isError, duration)` – generic helper for form/status messages.
  - `displayOwnedStalls()` – queries `stalls` where `ownerId` equals current user and lists them on the page.
  - `handleAddStallSubmit(event)` – validates the form, checks that the chosen stall ID is unique, then writes a new document in `stalls`.
  - `displayShopStatistics(ownedStallIds)` – counts responses for the owner’s stalls and displays totals and responses for today.
  - `initializeShopDashboard()` – orchestrates fetching stalls, statistics and hooking up the form once auth state is confirmed.

### 2.5 admin_dashboard.html + admin.js
* Available to users with role `admin`.
* Admin features:
  - Manage survey questions.
  - Manage users (promote/demote roles, remove user data).
  - View high level statistics and simple charts (Chart.js).
* Important functions from `admin.js`:
  - `fetchAndPopulateStalls()` – fills a dropdown with stall IDs for targeting questions.
  - `displayExistingQuestions()` – lists questions from Firestore and binds delete buttons.
  - `handleAddQuestionSubmit(event)` – validates and adds a question with selected scope (global or specific stalls).
  - `handleDeleteQuestion(event)` – removes a question document.
  - `displayUsers()` – lists all users with action buttons.
  - `handleChangeRole(userId, currentRole, action)` – updates a user document to promote/demote between `user`, `shop`, and `admin`.
  - `handleDeleteUser(userId, userEmail)` – deletes a user record from Firestore.
  - `calculateAndDisplayInsights()` – collects counts across users, stalls, questions and responses to populate metrics and a response trend chart. Utilises `renderResponseTrendChart(labels, data)` to draw the line graph using Chart.js.
  - `initializeAdminDashboard()` – loads stalls/questions/users then insights; sets up event listeners when the auth state confirms an admin user.

### 2.6 Shared Styling
* `style.css` – main stylesheet for all pages, around 1000 lines of rules for layout, typography and form components.
* `admin_style.css` is referenced but the consolidated styles appear in `style.css`.

## 3. Firestore Collections

The README describes the expected schema which is mirrored in the code:

- **users** `{uid, name, email, age, role, createdAt}`
- **questions** `{text, options[], scope('global'|'specific'), targetStalls[], createdAt, active}`
- **stalls** `{name, stallId, ownerId, location, createdAt}`
- **user_responses** `{userId, questionId, answer, stallId, submittedAt}`

Queries throughout the modules rely on these fields and assume indexes exist for composite conditions.

## 4. Key Flows

1. **User Registration & Login**
   - `auth.js` performs Firebase Authentication using email/password.
   - After login it fetches the user’s role from `users` collection and redirects to the appropriate dashboard.
   - Navigation elements on every page are updated by `auth.js` according to role and login state.

2. **Survey Participation**
   - `dashboard.js` reads `stall` parameter to load stall‑specific questions.
   - Already answered question IDs are fetched and new questions are rendered.
   - User selects an option; `handleAnswerSubmit` stores a document in `user_responses` with timestamp and stall ID.
   - Once all questions answered, the page shows a “thank you” message.

3. **Shop Owner Operations**
   - On shop dashboard initialisation, owned stalls are displayed via `displayOwnedStalls`.
   - The “Add Stall” form uses `handleAddStallSubmit` to create new stall entries after verifying uniqueness.
   - `displayShopStatistics` counts responses across the owner’s stalls (total and today) for quick insight.

4. **Administration**
   - Admin dashboard loads lists of stalls, users and questions on page load.
   - New questions can target all stalls or a selected subset.
   - `calculateAndDisplayInsights` aggregates counts for KPIs and uses Chart.js for a 7‑day response trend.
   - User management actions modify the `role` field or remove user data.

## 5. Important Utility Patterns

- **showStatus** – Reused across modules for unified success/error messaging.
- **Auth State Listeners** – Each page module waits for `auth.onAuthStateChanged` before initialising, preventing flash of unauthenticated content.
- **Firestore Queries with `where`/`or`** – `dashboard.js` performs compound queries to fetch global and stall specific questions in one call.
- **Chart.js Integration** – Admin dashboard charts are drawn inside `renderResponseTrendChart` and cleaned up on logout to free resources.

## 6. Future Considerations

- Backend Cloud Functions could move sensitive operations (e.g., user deletion) off the client.
- More comprehensive validation and error handling are needed before production deployment.
- The repository omits `auth.js` and `firebase_config.js` which must be supplied with real Firebase credentials for local testing.

---

This LLD reflects the state of the repository at commit time and summarises how each page and script work together to provide the survey workflow for Chai ThePeople.
