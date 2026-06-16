# Node.js AI SaaS Frontend

A production-grade Next.js 16 frontend for an AI-powered SaaS platform, built independently as part of a structured learning journey toward becoming an AI-integrated backend architect. Provides a complete UI for document management, RAG-based question answering, real-time embedding status updates, keyword search with highlighted results, usage tracking, and push notifications.

> **Note:** This is a portfolio/learning project. The private repository contains the full source code. This public repository contains documentation only.

---

## What This Project Demonstrates

- **Next.js 16 App Router** — route groups, protected routes via `proxy.js`, server and client components
- **Real-time UI** — Socket.IO client with JWT auth, document status updates without polling or page refresh
- **Push Notifications** — Firebase web push via native service worker (compatible with Firefox, Chrome, and Edge)
- **Search UX** — Elasticsearch full-text search with fuzzy matching, partial word matching, and highlighted snippets
- **RAG Interface** — Question answering with source attribution, confidence badges, and cache indicators
- **Auth Pattern** — JWT stored in localStorage + cookie, automatic redirect on 401, refresh token flow
- **Component Architecture** — Reusable stateless components, custom hooks, centralized API clients

---

## Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| Framework | Next.js 16 | React framework with App Router |
| Styling | Tailwind CSS 3 | Utility-first CSS |
| State | React Context | Global auth state |
| Real-time | Socket.IO client | WebSocket events from backend |
| Push | Firebase JS SDK v12 | FCM web push notifications |
| HTTP | fetch API | Custom API client with auth header |

---

## Pages

| Route | Description |
|---|---|
| `/login` | Login with email and password |
| `/register` | Register new account with auto-login |
| `/dashboard` | Usage stats, system health, recent documents and questions |
| `/documents` | Document list, upload, embed, delete, keyword search |
| `/ask` | RAG question answering with source cards and confidence badges |
| `/usage` | Token usage, AI breakdown, plan comparison, billing history |

---

## Key Features

### Real-time Document Status
Embedding status updates are pushed from the backend via Socket.IO — no polling. The document card status badge updates instantly when the embedding worker completes or fails.

```
User clicks Embed
  → API call queues BullMQ job
  → Optimistic UI update (status: embedding)
  → Backend worker processes job
  → Socket.IO emits document:status event to user's private room
  → Frontend receives event → updates React state
  → Status badge updates without page refresh
```

### Keyword Search
Elasticsearch full-text search with three match types:
- **Exact** — `saas` matches `SaaS`
- **Fuzzy** — `Elasitcsearch` matches `Elasticsearch` (typo tolerance)
- **Prefix** — `Elastic` matches `Elasticsearch` (partial word)

Results show highlighted snippets with matched terms wrapped in `<em>` tags.

### Push Notifications
Native service worker using the `push` event listener — not the Firebase compat SDK. This approach works across Firefox, Chrome, and Edge. The compat SDK causes empty `event.data` in Firefox.

```
User enables notifications in sidebar
  → Browser requests permission
  → Firebase SDK registers service worker
  → getToken() called with VAPID key
  → Token sent to backend POST /api/v1/users/fcm-token
  → Backend stores token in PostgreSQL
  → On usage page view → backend fires FCM notification
  → Service worker receives push event → shows notification
```

### Auth Flow
```
Login → store accessToken in localStorage + cookie
      → AuthContext reads localStorage on load
      → proxy.js checks cookie on every navigation
      → 401 response → clear storage → redirect to /login
      → Logout → clear localStorage + cookie + DELETE /users/fcm-token
```

---

## Component Structure

```
src/components/
├── Sidebar.jsx          # Navigation, notification bell, user info
├── DocumentCard.jsx     # Document with real-time status badge
├── SearchResultCard.jsx # Elasticsearch result with highlighted snippet
├── AnswerCard.jsx       # RAG answer with sources and confidence
├── SourceCard.jsx       # Individual source chunk reference
├── ConfidenceBadge.jsx  # High/medium/low confidence indicator
├── StatusBadge.jsx      # Document status pill (pending/embedding/embedded/failed)
├── StatCard.jsx         # Metric display card
├── PlanCard.jsx         # Billing plan with upgrade button
└── UploadDocumentModal.jsx  # Create document modal
```

---

## API Client Pattern

Each resource has its own API client in `src/lib/`:

```js
// src/lib/api.js — base client with auth header
export const api = {
    get: (path) => request('GET', path),
    post: (path, body) => request('POST', path, body),
    delete: (path) => request('DELETE', path),
};

// src/lib/documentsApi.js — resource-specific methods
export const documentsApi = {
    list: () => api.get('/documents'),
    create: (data) => api.post('/documents', data),
    embed: (id) => api.post(`/documents/${id}/embed`),
    delete: (id) => api.delete(`/documents/${id}`),
};
```

---

## Socket.IO Implementation Notes

The Socket.IO client connects directly to the backend IP rather than through the Next.js proxy — WebSocket upgrade requests are not reliably proxied by Next.js dev server.

On WSL2/Windows development:
- Use the WSL IP (`hostname -I | awk '{print $1}'`) not `localhost`
- Chrome uses WebSocket transport
- Firefox falls back to HTTP polling (Private Network Access restriction in dev)
- On production with a real domain and SSL — both browsers use WebSocket

The socket is a singleton — created once and kept alive across page navigations. Only disconnected on logout.

---

## Firebase Service Worker

`public/firebase-messaging-sw.js` uses the native Web Push API:

```js
self.addEventListener('push', (event) => {
    const data = event.data.json();
    const title = data.notification?.title;
    const body = data.notification?.body;
    event.waitUntil(
        self.registration.showNotification(title, { body })
    );
});
```

This approach is used instead of the Firebase compat SDK because the compat SDK produces empty `event.data` in Firefox.

---

## Project Versions

| Tag | Description |
|---|---|
| `v2.0-frontend` | Phase 4 — real-time updates, keyword search, push notifications, UI polish |

---

## Related Repository

Backend: [node-ai-saas-backend](https://github.com/kashifumar/node-ai-saas-backend)

---

## Author

**KASHIF UMAR**

[LinkedIn](https://www.linkedin.com/in/kashif-umar/) · [X](https://x.com/kashif_umar)

© 2025 All rights reserved. Unauthorized reproduction is not permitted.

---
