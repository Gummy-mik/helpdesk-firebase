
# Helpdesk (Firebase Simple, No-Install)

This is a **static** helpdesk web app that runs on **GitHub Pages** (or any static host) with **Firebase** for Auth + Database.

## What you get
- Login (Firebase Auth, email/password)
- Dashboard (Pending, Urgent, Completed this month — simple counts)
- Create Ticket (title/description/priority/branch)
- Tickets list (mine / assigned — simple filters)
- PM Planner (manual NextDueDate, mark completed)
- Firestore data model: `branches`, `tickets`, `pm_schedules`, `users`

## One-time setup (All in browser)
1. **Create Firebase project** → console.firebase.google.com
2. Enable **Authentication** → Sign-in method → **Email/Password** (enable)
3. Enable **Firestore** → Start in Test mode (then tighten rules below)
4. In Project settings → **Web App** → get your **firebaseConfig** object.
5. Open `public/js/firebaseConfig.js` and paste your config.
6. Push to GitHub (or upload). Enable **GitHub Pages** (Build from `main` → `/root`).
7. Visit your Pages URL (e.g., `https://<user>.github.io/<repo>/public/`).

## Firestore Security Rules (starter)
> Adjust emails/domains as needed. Limit write access to authenticated users only.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isSignedIn() { return request.auth != null; }

    match /users/{uid} {
      allow read: if isSignedIn();
      allow write: if isSignedIn() && request.auth.uid == uid;
    }

    match /branches/{id} {
      allow read: if true; // public read ok
      allow write: if false; // admin-only (tighten later)
    }

    match /tickets/{id} {
      allow read: if isSignedIn();
      allow create: if isSignedIn();
      allow update, delete: if isSignedIn(); // tighten by role later
    }

    match /pm_schedules/{id} {
      allow read: if isSignedIn();
      allow write: if isSignedIn(); // manual scheduling by signed-in users
    }
  }
}
```

## Collections and fields
- **branches**: { branchId (string), name, region, contactEmail }
- **tickets**: { title, description, priority, status, isUrgent, branchId, createdAt, createdByUid, assignedToUid }
- **pm_schedules**: { branchId, visitType, nextDueDate, lastServiceDate, vendor, isActive }
- **users**: { displayName, email, role, branchId }

## Roles (simple)
- requester, technician, supervisor, admin (store in `users.role`)

## Environment
Static only. Uses CDN Firebase v10 modules. No build tools needed.

