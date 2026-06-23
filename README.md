# ⚡ PulseFAQ — Crowd-Sourced FAQ Engine (Local JSON data layer)

Project scaffold for a MERN-like FAQ engine using a local JSON filesystem for persistence (no MongoDB). This repo contains a backend Express server, a Vite + React frontend with Tailwind CSS, and a `data/` folder for JSON storage. The application has been rebranded to **⚡ PulseFAQ** and includes an admin moderation desk and premium obsidian-themed UI.

**Structure**
- `server/` - Express API server (Node.js)
- `client/` - Vite + React + Tailwind frontend
- `data/` - JSON data files

**Data files**
- `data/faqs.json` - official approved FAQs (array of objects)
- `data/pending_aqs.json` - array of pending anonymous submissions

JSON item schemas
- Approved FAQ item (in `faqs.json`):

  {
    "id": "<string>",
    "question": "<text>",
    "answer": "<html/text>",
    "submission_count": 1,
    "isFAQ": true,
    "upvotes": 0,
    "downvotes": 0,
    "category": "optional category"
  }

- Pending AQ item (in `pending_aqs.json`):

  {
    "id": "<string>",
    "question": "<text>",
    "submission_count": 1,
    "isFAQ": false,
    "created_at": "ISO timestamp"
  }

**Server API** (default localhost:4000)

- `GET /api/faqs` — returns all items from `data/faqs.json`.
- `POST /api/aqs` — submit anonymous AQ. Body: `{ "question": "..." }`.
  - The server normalizes and tokenizes the incoming question, removes basic stopwords, and scans both `faqs.json` and `pending_aqs.json`. If a Jaccard similarity threshold (0.5) is met with an existing item, it increments that item's `submission_count` instead of creating a new record. Otherwise, it appends a new entry to `pending_aqs.json`.
- `PATCH /api/faqs/:id/vote` — vote on an approved FAQ. Body: `{ "vote": "up" }` or `{ "vote": "down" }`.

Admin endpoints (static credentials)
- `POST /api/admin/login` — Body `{ email, password }`. Successful credentials: `abc@gmail.com` / `samagama`. Responds with a static token string.
- `GET /api/admin/pending` — Requires header `x-admin-token: <token>`. Returns pending items for moderation.
- `POST /api/admin/approve` — Requires admin token. Body `{ id, answer }` — moves a pending item to `faqs.json` with the provided answer.
- `POST /api/admin/publish` — Requires admin token. Body `{ question, answer, category, isFAQ }` — publishes a new FAQ directly into `faqs.json` (used by the Manual FAQ Insertion desk).
- `DELETE /api/admin/pending` — Requires admin token. Body `{ ids: [id1, id2, ...] }` — bulk-delete pending items.
- `DELETE /api/admin/pending/:id` — Requires admin token. Deletes a single pending item by id.

Admin endpoints (static credentials)
- `POST /api/admin/login` — Body `{ email, password }`. Successful credentials: `abc@gmail.com` / `samagama`. Responds with a static token string.
- `GET /api/admin/pending` — Requires header `x-admin-token: <token>`. Returns pending items.
- `POST /api/admin/approve` — Requires admin token. Body `{ id, answer }` — moves pending item to `faqs.json` with provided answer.

**How file writes are handled**
- The server uses `fs.promises` to read and write JSON files directly. Reads verify file existence before parsing; writes serialize with `JSON.stringify(data, null, 2)` and atomically overwrite the target file. (This is a simple single-process filesystem-backed approach — for high concurrency or multi-process servers consider a proper database or file-locking mechanism.)

**Local setup**

1. Install dependencies for server and client separately.

Windows PowerShell commands (from repo root):

```
cd server; npm install
cd ../client; npm install
```

2. Run the server and client in separate terminals:

```
# terminal 1
cd server; npm run start

# terminal 2
cd client; npm run dev
```

Vite will run on `http://localhost:5173` and proxy `/api` to `http://localhost:4000` (configured in `client/vite.config.js`).

**Admin usage**
1. Open the client, use the Admin Login panel (right column). Enter `abc@gmail.com` / `samagama`.
2. On success the token is stored in `localStorage` and the Admin Dashboard displays pending submissions for moderation.

Frontend (Client) features — PulseFAQ Highlights
- Theme & Visuals: The UI uses an obsidian-style dark palette (deep charcoal backgrounds such as `bg-zinc-950` / `bg-slate-950`) with elevated card panels (`bg-zinc-900` / `bg-slate-900`) and subtle `border-zinc-800` accents. Interactive cards include a smooth hover scale/transition to provide a premium tactile feel.
- Manual FAQ Insertion desk: Admins have a dedicated "Manual FAQ Insertion" card in the Admin workspace where they can type a custom question, an official answer, and select a `category`. The form posts to `POST /api/admin/publish` and immediately refreshes the public FAQ list on success.
- Conditional Anonymous Submission: The anonymous "Crowd-Source an AQ" submission box is intentionally hidden whenever an admin is authenticated. This prevents admins from posting anonymous items while moderating and keeps the UI focused on moderation tools.
- Moderation tools: The Admin Moderation desk supports per-item inline answering, bulk approve (applies inline answers), and bulk delete of pending AQs. Selections clear after actions and the public feed refreshes automatically.

Voting & Interaction Pattern
- PulseFAQ implements a Reddit/StackOverflow-like voting interaction on each FAQ. Voting state is tracked locally in the browser to prevent duplicate votes per user using `localStorage` under the key `faq_votes` (an object mapping `{ [faqId]: 'upvote' | 'downvote' }`).
  - New vote: increments the chosen counter locally and sends a `PATCH /api/faqs/:id/vote` request to increment the same counter server-side.
  - Toggle (same vote again): removes the local vote (decrements the local counter) and clears the stored vote. Because the backend currently supports increment-only PATCH operations, removals are handled client-side for immediate UX consistency.
  - Switch vote: decrements the previous local counter, increments the new local counter, updates `localStorage`, and sends a single `PATCH` to increment the new vote on the server.
  - This approach provides an optimistic, fast UI while maintaining server-side increment telemetry; for fully authoritative vote counts consider adding decrement-capable endpoints or moving votes into a transactional datastore.

**Notes & Next Steps**
# Notes & Next Steps
- This is a filesystem-backed prototype. For production, replace the static admin token with a secure auth flow, and consider a proper DB for concurrency and transactional vote handling.
- The keyword similarity threshold (0.5 Jaccard) is configurable in `server/routes/faqs.js`.
# team-6a314e2012663badc7eb1814
FAQ Crowdsourcing project — Samriddhi Bansal