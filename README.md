# Wattpad-like App Plan and Mega Prompt

## Pasos esenciales (resumen)

1. **Define el stack**: Next.js + TypeScript + Tailwind + shadcn/ui + Prisma + PostgreSQL + Auth.js (OAuth/Email) + Stripe (suscripciones opcional) + Cloudinary/S3 (portadas) + next-pwa (offline) + Zod (validación) + React Hook Form + Framer Motion (animaciones).
2. **Modela datos** (User, Story, Chapter, Tag, StoryTag, Comment, Like, Follow, ReadingList, Notification, Report, View, Rating, Message/Thread opcional, Subscription).
3. **Flujos base**: registro/login, crear historia, editor con autosave, publicar capítulos, portada y tags, descubrimiento (feed, tendencias, búsqueda), lectura con progreso guardado, comentarios/likes, seguir autores, listas de lectura.
4. **UI/UX**: tema claro/oscuro, tipografías de lectura, controles (tamaño de letra/espaciado/margen), paginado/scroll, “Continuar leyendo”, atajos de teclado, skeletons, toasts, microinteracciones.
5. **Búsqueda y ranking**: texto completo (Postgres FTS o Meilisearch), orden por relevancia/reciente/ratio de lectura/completadas.
6. **Moderación/seguridad**: reportes, panel admin, filtros NSFW + clasificación por edades, bloqueo de usuarios, subida segura, Terms/Privacy (GDPR UE).
7. **PWA & móvil**: service worker, caché para lectura offline de capítulos guardados, instalable en iOS/Android.
8. **Analítica**: vistas, lecturas completadas, retención capítulo-a-capítulo, conversión a seguidores/suscriptores (gráficas).
9. **Monetización (opcional)**: propinas/suscripciones con Stripe, capítulos premium, anuncios con “ad slots” desactivables para premium.
10. **Deploy**: DB (Neon/Supabase/Render Postgres), imágenes (Cloudinary o S3), web (Vercel), variables de entorno, semilla de datos y README.

---

## SUPER PROMPT PARA CODEX

```
You are a senior full-stack engineer. Generate a production-ready Wattpad-like web app (UGC reading/writing platform) with a modern, beautiful UI and complete feature set. Use the following spec and deliver a working monorepo with clear instructions.

=== PROJECT GOAL ===
A platform where users read and publish stories with chapters, interact via comments/likes/follows, discover content via feeds and search, save reading progress, and use an editor with autosave. Include PWA offline reading, analytics with charts, admin moderation tools, and optional subscriptions.

=== TECH STACK ===
- Framework: Next.js (App Router) + TypeScript
- Styling: Tailwind CSS + shadcn/ui components + Radix primitives + Framer Motion for micro-animations
- Forms/Validation: React Hook Form + Zod
- Auth: Auth.js (email/password + OAuth: Google, GitHub). Email verification and password reset flows.
- DB: PostgreSQL + Prisma ORM (with migrations & seed)
- Search: PostgreSQL Full-Text Search (tsvector/tsquery) with indexes (or Meilisearch adapter behind a feature flag)
- Uploads: Cloudinary (images) with signed uploads
- PWA: next-pwa, installable app, offline cache for saved chapters (“Download for offline”)
- Payments (optional): Stripe for tips/subscriptions (creator support, premium chapters)
- Emails: React Email + Resend (feature-flag)
- Analytics: simple in-app analytics stored in DB + charts with Chart.js
- Lint/Format/Test: ESLint + Prettier + vitest + testing-library/react
- DevX: Turborepo monorepo; packages: ui, config, eslint; app in apps/web
- Deploy: Vercel (web), Neon/Supabase (Postgres), Cloudinary (media). Provide .env.example and a full README with deploy steps.

=== DESIGN SYSTEM & UX ===
- Include a polished, responsive design with light/dark theme toggle.
- Typography optimized for reading (serif option for reading view).
- Components: Button, Input, Textarea, Select, Badge, Card, Modal/Dialog, Drawer (mobile), Tabs, Tooltip, Dropdown, Avatar, Menu, Breadcrumbs, Toast, Pagination, InfiniteScroll, Skeleton, EmptyState, Tag/Chip, Stepper (tutorial).
- Animations: subtle hover/tap, page transitions, like/bookmark confetti micro-animation, skeleton shimmer.
- Decorative visuals: soft gradients and glassmorphism for hero/landing. Include an illustrations folder with 2–3 placeholder SVGs.

=== DATA MODEL (Prisma) ===
- User { id, username unique, name, email unique, bio, avatarUrl, headerUrl, role enum: USER|AUTHOR|MOD|ADMIN, createdAt }
- Story { id, authorId FK, title, slug unique, synopsis, coverUrl, status enum: DRAFT|ONGOING|COMPLETED|HIATUS, rating enum: G|T|M|E (age), isNsfw boolean, wordsCount, readsCount, likesCount, bookmarksCount, publishedAt, updatedAt }
- Chapter { id, storyId FK, index int, title, content (Markdown), wordsCount, isPublished, publishedAt }
- Tag { id, name unique }
- StoryTag { storyId, tagId } composite PK
- Comment { id, storyId FK, chapterId FK nullable, authorId FK, content, createdAt, parentId nullable (threaded) }
- Like { id, userId FK, storyId FK, createdAt } (unique compound)
- Follow { followerId, followingId, createdAt } (unique compound)
- ReadingList { id, userId FK, name, isPrivate; ReadingListItem { listId, storyId, createdAt } }
- Progress { id, userId, storyId, chapterId, percent float, updatedAt }
- Notification { id, userId, type enum, payload jsonb, isRead, createdAt }
- Report { id, reporterId, targetType enum STORY|COMMENT|USER, targetId, reason enum, details, status enum OPEN|REVIEW|CLOSED, createdAt }
- View { id, storyId, chapterId nullable, userId nullable, userAgent, createdAt }
- Rating { id, storyId, userId, stars int 1–5, createdAt }
- MessageThread { id, title nullable, createdAt }
- MessageParticipant { threadId, userId, role enum OWNER|MEMBER }
- Message { id, threadId, senderId, content, createdAt }
- Subscription (Stripe) { id, userId, stripeCustomerId, stripeSubId, status, createdAt }

Include necessary indexes (e.g., Story.slug, text search indexes for Story.title/synopsis and Chapter.content; composite uniques for Like/Follow/ReadingListItem; foreign keys with cascading behavior where safe).

=== CORE FEATURES ===
1) **Auth & Profiles**
   - Sign up/login (email & OAuth), email verification, reset password.
   - Public profile: avatar, bio, header, stats (stories, followers, total reads, avg rating), follow/unfollow.

2) **Story Creation**
   - “New Story” flow with title, synopsis, cover (crop), tags, rating/NSFW, status.
   - Editor for chapters: Markdown with toolbar (bold/italic/headers/links/images), autosave drafts, preview, word count, reorder chapters via drag & drop, schedule publish, draft vs published.
   - Story dashboard (author): manage chapters, analytics (reads per chapter, drop-off, likes/bookmarks), comments moderation, visibility.

3) **Reading Experience**
   - Reader page focused: font family/size, line-height, margins, theme, page/scroll mode, estimated time.
   - Save progress automatically; “Continue Reading” widget on home/profile.
   - Offline: user can “Save for offline” a story/chapters (PWA cache). Show offline badge when available.

4) **Engagement**
   - Likes, comments (threaded, reply/mention), bookmarks, reading lists personalizadas (públicas/privadas), follow autores, notifications (new chapter, new comment, new follower).
   - Share (Open Graph cards), copy link, deep-links a capítulo.
   - Report content/users; block/mute.

5) **Discovery & Search**
   - Home feed: For You (personalized by follows/tags/history), Trending (by views velocity/likes/bookmarks, time-decayed), New & Noteworthy, Editors’ Picks (admin curated).
   - Explore por tags/genres, filtros (status, rating, nsfw toggle, length, language).
   - Search con FTS across stories/chapters/users. Highlight matches.

6) **Lists & Collections**
   - Reading Lists: create/rename/reorder; add/remove; shareable public page.
   - “Continue Reading”, “Saved Offline”, “Recent”.

7) **Messaging (opcional)**
   - Simple DM threads (abide by safety). Users can disable DMs.

8) **Admin/Moderation**
   - Admin panel: user/story management, reports queue (triage, status), tag management, featured slots, site banners, editorial collections.
   - Analytics dashboards with charts: DAU/MAU, reads per day, retention by chapter, conversion to follow, top genres/tags.

9) **Monetización (opcional)**
   - Stripe: creator tips and/or subscriptions tiered (ad-free, early access, premium chapters). Gating via middleware. Webhooks for Stripe events.

10) **Internationalization & Accessibility**
   - i18n (en/es) with default es-ES; all copy externalized.
   - A11y: semantic HTML, keyboard navigation, ARIA, color-contrast checks.

=== PAGES/ROUTES (Next.js App Router) ===
- / (Landing + feed For You/Trending + Continue Reading)
- /explore (tags/filters)
- /search?q=
- /auth/(sign-in|sign-up|reset|verify)
- /u/[username] (profile) + /u/[username]/followers + /u/[username]/lists
- /story/new, /story/[slug], /story/[slug]/edit
- /story/[slug]/chapter/[index]
- /lists, /lists/new, /lists/[id]
- /notifications
- /messages, /messages/[threadId] (feature flag)
- /admin (overview, reports, users, stories, tags, featured, analytics)
- /settings (profile, account, notifications, privacy)
- /tos, /privacy, /guidelines

=== API ENDPOINTS (REST or server actions) ===
Auth, Users, Stories, Chapters (CRUD + publish/schedule), Tags, Comments (threaded), Likes, Follows, ReadingLists, Progress, Search, Notifications, Reports, Views (track), Analytics (aggregate queries), Stripe webhooks.

=== PWA & OFFLINE DETAILS ===
- Cache strategy: static assets (stale-while-revalidate), chapters saved by user (Cache Storage per story), fallback read view when offline.
- Install prompt + icons + splash screens. “Available offline” indicator and management page to remove downloads.

=== ANALYTICS ===
- Record View events on story/chapter with server action (deduplicate via cookie/uid window).
- Dashboards: charts for reads/day, completion funnel, retention across chapters, top tags. Use Chart.js and card widgets.

=== MODERATION & SAFETY ===
- Report flows with reasons (spam, hate, sexual content involving minors, etc.). Queue in admin. Soft delete with tombstones.
- NSFW flag + age rating required; blur covers for NSFW when enabled. Parental warning gates.
- GDPR: cookie consent banner, data export/delete account, privacy policy page.

=== UI POLISH ===
- Hero landing with headline, sample screenshots, animated gradient background, call-to-action.
- Story cards with cover, tag chips, status badge, stats; hover elevate.
- Reader HUD: minimal controls bar; keyboard shortcuts (←/→ chapters, +/- font size, T theme).
- Toasts and skeleton loaders for all async flows.

=== DEV & DEPLOY ===
- Provide: .env.example, prisma schema, migration, seed script (create test users, tags, 10 sample stories with 5+ chapters each).
- README with: setup, env vars, running dev (db, web), build, deploy to Vercel + Neon + Cloudinary, Stripe setup notes, Resend notes.
- Dockerfile + optional docker-compose.yml for local Postgres/Meilisearch.
- CI: GitHub Actions for lint/test/build.
- Code quality: strict TS, ESLint/Prettier configured.

=== EXTRA REQUIREMENTS ===
- Comprehensive error handling and empty states.
- Security: rate limiting (simple token bucket on write endpoints), input sanitation, upload validation, RLS-like checks in code.
- Performance: image optimization, incremental static generation for public pages where possible, pagination & infinite scroll, DB indexes.

Deliver the complete codebase with the structure:
- /apps/web (Next.js)
- /packages/ui (shared components)
- /packages/config, /packages/eslint
- /prisma (schema, seeds)
Include sample assets (svg illustrations), and ensure the app boots with mocked envs and seed data in dev mode.

Output:
1) File tree
2) Key code files in full (pages, components, prisma schema, major API routes, PWA config)
3) Seed script
4) README with detailed instructions
```

---

¿Quieres también una versión móvil nativa (Expo/React Native) del lector con sincronización de progreso, o prefieres comenzar con la PWA?
