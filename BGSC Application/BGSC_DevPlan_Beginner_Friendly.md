# BGSC Platform — Beginner-Friendly Development Plan
**Version:** 1.0 | **Last Updated:** June 14, 2026 | **Critical Deadline:** Backend MVP → June 26, 2026

---

## 🧭 How to Use This Document

This plan is written for developers who may be new to microservices, NestJS, or large-scale projects. Every section answers three questions:

- **What** do you build?
- **Why** does it exist?
- **How** do you know you've done it right?

Read each section top-to-bottom before touching code. Ask questions in `#dev-help` before going down a rabbit hole.

---

## ⚡ The One Thing You Must Know Immediately

```
Backend MVP hard deadline: JUNE 26, 2026
Frontend MVP follows:       June 27 – July 10, 2026
Today is:                   June 14, 2026
Time remaining:             12 days for all backend work
```

Every task is assigned a **priority badge**:

| Badge | Meaning |
|-------|---------|
| 🔴 MUST | Blocks the June 26 deadline. Do these first. |
| 🟡 SHOULD | Important but can slip by one day if necessary. |
| 🟢 CAN | Nice to have; defer to Phase 2 if under pressure. |

---

## 📦 What Are We Building?

BGSC Platform is a **mobile + web app** for BITS Goa's sports and esports communities. Think of it as a college sports hub that combines:

- **Event discovery & registration** (like BookMyShow, but for college tournaments)
- **Social feed & friends** (like Instagram, but campus-only)
- **Points & gamification** (users earn points and affiliate with sponsors)
- **Internal ops tool** (a Notion/Jira hybrid for the organizing crew — the "Union Page")

There are two client apps sharing one backend:

| App | Who Uses It | Built With |
|-----|-------------|------------|
| **Mobile App** (iOS/Android) | Students — register for events, post, earn points | React Native + Expo |
| **Web Admin Console** (PWA) | Coordinators — create events, manage leagues, run auctions | React + Tailwind CSS |

---

## 👶 Before You Write a Single Line of Code

### Required installs (do these on Day 1)

```bash
# 1. Node.js 20+ (LTS)
node --version   # Should print v20.x.x or higher

# 2. Docker Desktop
docker --version

# 3. Git
git --version

# 4. Expo CLI (for mobile dev)
npm install -g expo-cli

# 5. NestJS CLI (for backend services)
npm install -g @nestjs/cli

# 6. Postman (API testing) — download from postman.com

# 7. VS Code extensions (install from VS Code marketplace):
#    - ESLint
#    - Prettier
#    - REST Client
#    - GitLens
```

### Required accounts (make these before Day 1)

- **GitHub** — to access the `bgsc-platform` repo
- **Railway** — free tier, for hosting PostgreSQL + Redis + backend during development
- **Sentry** — free tier, for error tracking
- **Cloudinary** — free tier, for image uploads in MVP

### Foundational knowledge checklist

If any of these are unfamiliar, spend 30–60 minutes on each *before* starting that phase:

| Topic | Needed for | Free Resource |
|-------|-----------|---------------|
| REST APIs (what are GET/POST/PATCH?) | Phase 0 | [MDN HTTP Methods](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods) |
| JWT authentication | Phase 0 | [jwt.io Introduction](https://jwt.io/introduction) |
| TypeScript basics | Phase 0 | [TypeScript in 5 minutes](https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes.html) |
| NestJS fundamentals | Phase 0 | [NestJS First Steps](https://docs.nestjs.com/first-steps) |
| PostgreSQL + SQL basics | Phase 0 | [PostgreSQL Tutorial](https://www.postgresqltutorial.com) |
| React Native basics | Phase 1 Frontend | [Expo Tutorial](https://docs.expo.dev/tutorial/introduction/) |
| Docker basics | Phase 0 | [Docker Get Started](https://docs.docker.com/get-started/) |

---

## 🗺️ Plain-English Tech Stack

You'll see many acronyms and tool names throughout this document. Here's what each one actually does:

| Tool | Plain English | Used For |
|------|--------------|----------|
| **NestJS** | A framework that makes Node.js backend code organized and scalable. Think of it like Express.js but with structure enforced. | All backend microservices |
| **PostgreSQL** | The main database. Stores users, events, points — anything with relationships. | Primary data store |
| **Redis** | A super-fast in-memory store. Not for permanent data — used for sessions, rate limits, caches that can be regenerated. | Sessions, caches, real-time |
| **JWT** | A signed token (like a wristband at a concert) that proves who a user is. Contains their user ID and role. | Authentication |
| **React Native + Expo** | React, but it compiles to iOS and Android apps. Expo is the toolchain that makes it easy to build and test. | Mobile app |
| **Kafka** | A "message bus" — services publish events, other services subscribe and react. Introduced in Phase 2; for MVP we use a simple alternative. | Event-driven comms (Phase 2+) |
| **EventEmitter** | Node's built-in simple pub/sub mechanism. We use it instead of Kafka in Phase 0–1. | Event-driven comms (MVP only) |
| **Docker Compose** | A file that lets you start all services (database, redis, backend) with one command: `docker-compose up`. | Local dev environment |
| **GitHub Actions** | Automated scripts that run tests and deploy code whenever you push to GitHub. | CI/CD pipeline |
| **Swagger** | Auto-generates a web UI from your API so you can test endpoints in a browser. NestJS generates this automatically. | API docs + manual testing |
| **Prisma** | An ORM — it lets you write TypeScript to query the database instead of raw SQL. | Database access |
| **React Query (TanStack)** | On the frontend, handles fetching data from APIs, caching, and re-fetching when stale. | Frontend data fetching |
| **Zustand** | Lightweight global state management for React/React Native. Simpler than Redux. | Frontend global state |
| **MVVM** | Model-View-ViewModel — an architecture pattern. Model = data, View = UI components, ViewModel = the glue (state + logic) between them. | Frontend architecture |
| **RBAC** | Role-Based Access Control — users have roles (Guest, User, Member, Core, Coordinator, Founder), and each role has different permissions. | Authorization |

---

## 🏗️ Codebase Structure (Monorepo)

```
bgsc-platform/
├── backend/                  # All NestJS microservices
│   ├── apps/
│   │   ├── auth/             # Handles login, register, JWT
│   │   ├── user/             # User profiles, RBAC
│   │   ├── event/            # Events, registration, leaderboards
│   │   ├── points/           # Points balance, transactions
│   │   ├── sponsor/          # Sponsor CRUD, fan counting
│   │   ├── notification/     # In-app notifications
│   │   └── hall-of-fame/     # Winners archive
│   ├── libs/
│   │   ├── common/           # Shared DTOs, guards, decorators
│   │   ├── event-bus/        # In-memory EventEmitter wrapper (Phase 1)
│   │   └── database/         # Prisma client, migrations
│   └── docker-compose.yml    # Spin up postgres + redis + services locally
│
├── mobile/                   # React Native (Expo) app
│   ├── src/
│   │   ├── screens/          # One folder per screen
│   │   ├── components/       # Reusable UI components
│   │   ├── viewmodels/       # MVVM ViewModels (state + actions)
│   │   ├── repositories/     # API call functions
│   │   └── store/            # Zustand global state
│   └── app.json              # Expo config
│
├── web/                      # React + Tailwind Admin PWA
│   ├── src/
│   │   ├── pages/            # Admin screens
│   │   ├── components/       # UI components
│   │   └── api/              # API client functions
│   └── vite.config.ts
│
└── .github/
    └── workflows/            # GitHub Actions CI/CD files
```

> **💡 New to the repo?** The first thing to do is run `docker-compose up` from the `backend/` folder. All services start automatically.

---

## 📅 Timeline at a Glance

| Phase | Focus | Dates | Status |
|-------|-------|-------|--------|
| **Phase 0** | Foundation: infra, auth, shell | Jun 14 – Jun 19 | 🔴 IN PROGRESS |
| **Phase 1 Backend** | All backend APIs for MVP | Jun 14 – Jun 26 | 🔴 URGENT |
| **Phase 1 Frontend** | Mobile + Web consuming the APIs | Jun 27 – Jul 10 | 🟡 UPCOMING |
| **Phase 2** | Friends, posts, challenges, Kafka | Jul 11 – Aug 21 | ⬜ PLANNED |
| **Phase 3** | Union Page, leagues, auctions | Aug 22 – Oct 16 | ⬜ PLANNED |
| **Phase 4** | Integrations + polish | Oct 17 – Nov 13 | ⬜ PLANNED |
| **Phase 5** | Buffer, load testing, launch | Nov 14 – Dec 11 | ⬜ PLANNED |

> **⚠️ Phase 0 and Phase 1 Backend overlap.** Phase 0 infrastructure work (repo setup, Docker, CI/CD) happens in the first 5 days. Phase 1 backend services start as soon as infrastructure is ready.

---

## 🏁 Phase 0: Foundation (June 14–19)

**Goal:** Get everyone set up, nothing user-facing yet. After Phase 0, any developer can clone the repo, run `docker-compose up`, and have a working local environment.

---

### Milestone 0.1 — Repo & CI/CD 🔴 MUST

> **What is this?** A GitHub repository everyone shares. CI/CD means tests and deployments run automatically on every push — so you catch broken code before it reaches teammates.

#### Tasks

| Task | Steps | Who | Time |
|------|-------|-----|------|
| Create `bgsc-platform` GitHub repo | Create as org repo; add all team members | DevOps lead | 1 hr |
| Branch protection | In repo Settings → Branches: protect `main` (require PR + 1 review, no direct push). Protect `dev` (require CI to pass). | DevOps lead | 30 min |
| Add `.gitignore` | Use [GitHub's Node.js gitignore template](https://github.com/github/gitignore/blob/main/Node.gitignore). Add `.env` to it. | Anyone | 15 min |
| GitHub Actions — backend lint & test | Create `.github/workflows/backend.yml` (see starter below) | DevOps lead | 2 hr |
| GitHub Actions — Docker build on PR | Add `docker/build-push-action` step to the workflow | DevOps lead | 1 hr |

**GitHub Actions starter (`backend.yml`):**

```yaml
name: Backend CI
on:
  push:
    branches: [dev, main]
  pull_request:
    branches: [dev, main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json
      - run: cd backend && npm ci
      - run: cd backend && npm run lint
      - run: cd backend && npm test
```

**✅ How to know this is done:**
- Push any change to `dev` → GitHub Actions tab shows a green checkmark.
- Creating a PR to `main` without a review → GitHub blocks the merge.

---

### Milestone 0.2 — Staging Environment 🔴 MUST

> **What is this?** A cloud environment where the app lives so the team can test without running code locally all the time. We use Railway (free tier, no credit card needed).

#### Tasks

| Task | Steps | Time |
|------|-------|------|
| Create Railway project | Go to railway.app → New Project → provision **PostgreSQL** and **Redis** plugins | 30 min |
| Note connection strings | Railway gives you `DATABASE_URL` and `REDIS_URL`. Put them in a team-shared `.env.staging` (never commit this file). | 15 min |
| Set up Sentry | Create a new project at sentry.io → choose Node.js → copy the DSN string | 30 min |
| Set up Cloudinary | Create account at cloudinary.com → copy `cloud_name`, `api_key`, `api_secret` | 15 min |

**✅ How to know this is done:**

```bash
# From your terminal, test that the staging DB is reachable:
psql "your-railway-postgres-url" -c "SELECT version();"
# Should print: PostgreSQL 15.x ...
```

---

### Milestone 0.3 — Backend Core: Auth & User Services 🔴 MUST

> **What is this?** The most fundamental backend piece. The Auth Service lets people sign up and log in. The User Service stores their profile and role. Everything else depends on these.

#### Setting up the NestJS monorepo

```bash
# One-time setup for the entire backend monorepo
nest new backend --package-manager npm
cd backend
# Add NestJS monorepo support
nest generate app auth
nest generate app user
nest generate library common
nest generate library database
```

#### Auth Service tasks

| Task | NestJS Commands & Notes | Time |
|------|------------------------|------|
| **Install dependencies** | `npm install @nestjs/jwt @nestjs/passport passport passport-jwt passport-google-oauth20 bcrypt` | 20 min |
| **`POST /auth/register`** | Accept `{ username, email, password, sponsorId }`. Hash password with bcrypt (10 rounds). Create user in DB. Return JWT pair. | 3 hr |
| **`POST /auth/login`** | Accept `{ email, password }`. Verify password. Return JWT access token (15 min TTL) + refresh token (7 day TTL, stored in Redis). | 2 hr |
| **`POST /auth/refresh`** | Accept refresh token in header. Verify in Redis. Issue new access token. Rotate refresh token. | 1 hr |
| **`POST /auth/logout`** | Delete refresh token from Redis. Return 200. | 30 min |
| **`POST /auth/google`** | OAuth2 callback. Create user if first login, otherwise return JWT. | 2 hr |
| **`POST /auth/forgot-password`** | (Stub for MVP — just return 200 and log the email. Implement email in Phase 2.) | 30 min |

**JWT setup example:**

```typescript
// backend/apps/auth/src/auth.module.ts
@Module({
  imports: [
    JwtModule.registerAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        secret: config.get('JWT_SECRET'),
        signOptions: { expiresIn: '15m' },
      }),
    }),
  ],
})
export class AuthModule {}
```

**What goes in a JWT payload?**

```typescript
// Keep it small — only what every service needs
interface JwtPayload {
  sub: string;      // user ID
  role: UserRole;   // guest | user | member | core | coordinator | founder
  iat: number;      // issued at (auto-added by NestJS)
  exp: number;      // expiry (auto-added)
}
```

#### User Service tasks

| Task | Notes | Time |
|------|-------|------|
| **Prisma schema for `users` table** | See schema below | 1 hr |
| **`GET /users/me`** | Return current user's profile (from JWT). | 1 hr |
| **`GET /users/:id`** | Return public fields only (username, avatar, bio, interests, sponsor badge). Never return email/phone to non-friends. | 1 hr |
| **`PATCH /users/me`** | Update bio, interests, social links, avatar URL. | 1 hr |
| **RBAC `RolesGuard`** | NestJS guard that checks `req.user.role` against `@Roles(...)` decorator. | 2 hr |

**Prisma schema (`libs/database/schema.prisma`):**

```prisma
enum UserRole {
  GUEST
  USER
  MEMBER
  CORE
  COORDINATOR
  FOUNDER
}

model User {
  id              String   @id @default(cuid())
  username        String   @unique
  email           String   @unique
  passwordHash    String?
  contact         String?
  role            UserRole @default(USER)
  avatarUrl       String?
  bio             String?
  interests       String[] // e.g. ["football", "valorant"]
  pointsBalance   Int      @default(0)
  activeSponsorId String?
  createdAt       DateTime @default(now())
  lastActive      DateTime @updatedAt

  sponsor         Sponsor? @relation(fields: [activeSponsorId], references: [id])
  registrations   Registration[]
  pointTransactions PointTransaction[]
}
```

```bash
# Run migrations
npx prisma migrate dev --name init
```

#### RBAC Guard (shared across all services)

```typescript
// libs/common/src/guards/roles.guard.ts
@Injectable()
export class RolesGuard implements CanActivate {
  constructor(private reflector: Reflector) {}

  canActivate(context: ExecutionContext): boolean {
    const requiredRoles = this.reflector.getAllAndOverride<UserRole[]>('roles', [
      context.getHandler(),
      context.getClass(),
    ]);
    if (!requiredRoles) return true; // No roles needed = public endpoint

    const { user } = context.switchToHttp().getRequest();
    return requiredRoles.some(role => this.hasPermission(user.role, role));
  }

  private roleHierarchy = ['GUEST', 'USER', 'MEMBER', 'CORE', 'COORDINATOR', 'FOUNDER'];
  
  private hasPermission(userRole: UserRole, requiredRole: UserRole): boolean {
    return this.roleHierarchy.indexOf(userRole) >= this.roleHierarchy.indexOf(requiredRole);
  }
}

// Usage on any controller:
@Get('admin/users')
@Roles(UserRole.COORDINATOR)  // COORDINATOR and above can access this
@UseGuards(JwtAuthGuard, RolesGuard)
getUsers() { ... }
```

**✅ Phase 0 success checklist:**
- [ ] `docker-compose up` starts postgres + redis + auth service + user service without errors
- [ ] `POST /auth/register` → returns `{ accessToken, refreshToken, user }`
- [ ] `POST /auth/login` with wrong password → returns 401
- [ ] `GET /users/me` with valid JWT → returns user data
- [ ] `GET /users/me` without JWT → returns 401
- [ ] Unit test coverage > 50% on auth and user services (`npm test`)
- [ ] GitHub Actions pipeline passes on a new PR

---

### Milestone 0.4 — Event Bus (In-Memory) 🟡 SHOULD

> **What is this?** Services need to "talk" to each other. For example, when a user registers for an event, the Points service needs to know so it can award points. We use a simple Node.js EventEmitter for MVP. Kafka (the "proper" version) comes in Phase 2.

```typescript
// libs/event-bus/src/event-bus.service.ts
import { Injectable } from '@nestjs/common';
import { EventEmitter } from 'events';

@Injectable()
export class EventBusService {
  private emitter = new EventEmitter();

  publish(eventName: string, payload: object): void {
    this.emitter.emit(eventName, payload);
  }

  subscribe(eventName: string, handler: (payload: any) => void): void {
    this.emitter.on(eventName, handler);
  }
}

// How to use in a service:
// PUBLISHING (Event service, after registration)
this.eventBus.publish('RegistrationCreated', { userId, eventId, timestamp: new Date() });

// SUBSCRIBING (Points service, to award points)
this.eventBus.subscribe('RegistrationCreated', async ({ userId, eventId }) => {
  await this.awardPoints(userId, 10, 'event_registration', eventId);
});
```

---

### Milestone 0.5 — Mobile Shell 🟡 SHOULD

> **What is this?** The empty app skeleton. No real data yet — just navigation, the status bar, theme switching, and empty placeholder screens. This lets the mobile team set up their environment before Phase 1 APIs are ready.

```bash
# Create Expo project
npx create-expo-app mobile --template blank-typescript
cd mobile
npx expo install react-native-gesture-handler react-native-reanimated @react-navigation/native
npm install zustand @tanstack/react-query
```

**Navigation structure to build now:**

```
<NavigationContainer>
  <Drawer.Navigator>           ← Side navigation drawer
    <Drawer.Screen name="Home" />
    <Drawer.Screen name="Events" />
    <Drawer.Screen name="Leaderboards" />
    <Drawer.Screen name="HallOfFame" />
    <Drawer.Screen name="Points" />
    <Drawer.Screen name="Sponsors" />
    <Drawer.Screen name="Friends" />
    <Drawer.Screen name="Store" />
    <Drawer.Screen name="Media" />
    <Drawer.Screen name="Feedback" />
  </Drawer.Navigator>
</NavigationContainer>
```

---

## 🚀 Phase 1: Backend MVP (June 14–26) — URGENT

> **Context for beginners:** Everything in this section must be done and deployed by June 26. The frontend team starts building on June 27 and they need working APIs. If an API isn't ready, the frontend can't proceed.

### Working agreement for Phase 1

1. **API contracts first.** Before writing a line of code for a new endpoint, define it in Swagger (NestJS makes this easy with `@ApiOperation`, `@ApiBody`, etc.) and share with the mobile team. They can build mock screens while the real API is being coded.
2. **One service per person.** Assign each person one or two services so there's no overlap.
3. **Daily check-in.** Quick 10-minute standup: what's done, what's blocked.
4. **Test everything with Postman.** Export a shared Postman collection and update it whenever an endpoint is added or changed.

---

### Milestone 1.1 — Sponsor System v1 🔴 MUST

> **What is a Sponsor?** Real companies that "sponsor" the BGSC community. Users pick one sponsor during onboarding. When a user wins an event, they earn "fans" (virtual points) for their sponsor. Sponsors compete on a leaderboard.

#### Data model additions

```prisma
model Sponsor {
  id            String   @id @default(cuid())
  name          String
  logoUrl       String
  description   String?
  websiteUrl    String?
  tenureStart   DateTime
  tenureEnd     DateTime
  status        SponsorStatus @default(ACTIVE)
  totalFans     Int      @default(0)
  users         User[]
  affiliations  UserSponsorAffiliation[]
}

model UserSponsorAffiliation {
  id          String   @id @default(cuid())
  userId      String
  sponsorId   String
  affiliatedAt DateTime @default(now())
  fanCount    Int      @default(0)
  semester    String   // e.g. "2026-S1"
  user        User     @relation(...)
  sponsor     Sponsor  @relation(...)
  @@unique([userId, semester])  // one sponsor per semester per user
}

enum SponsorStatus {
  ACTIVE
  INACTIVE
}
```

#### Endpoints to build

| Endpoint | Auth | What it does |
|----------|------|-------------|
| `GET /sponsors/active` | Public | List all active sponsors for current semester |
| `GET /sponsors/leaderboard` | Public | Sponsors sorted by fan count |
| `POST /sponsors` | Coordinator+ | Create a new sponsor |
| `PATCH /sponsors/:id` | Coordinator+ | Update sponsor details |
| `POST /users/me/sponsor` | User | Select/change sponsor (once per semester) |
| `GET /users/me/sponsor-stats` | User | My fan count, events won for my sponsor |

**Semester change limit logic:**

```typescript
async changeSponsor(userId: string, newSponsorId: string) {
  const currentSemester = this.getCurrentSemester(); // e.g. "2026-S1"
  
  const existingAffiliation = await this.prisma.userSponsorAffiliation.findUnique({
    where: { userId_semester: { userId, semester: currentSemester } }
  });

  if (existingAffiliation) {
    throw new BadRequestException('You can only change sponsors once per semester.');
  }

  return this.prisma.userSponsorAffiliation.create({
    data: { userId, sponsorId: newSponsorId, semester: currentSemester }
  });
}

private getCurrentSemester(): string {
  const now = new Date();
  const year = now.getFullYear();
  const semester = now.getMonth() < 6 ? 'S1' : 'S2';
  return `${year}-${semester}`;
}
```

**✅ Done when:**
- `GET /sponsors/active` returns a list of sponsors
- `POST /users/me/sponsor` works, and calling it twice in the same semester returns 400

---

### Milestone 1.2 — Events & Registration 🔴 MUST

> **What is an Event?** Any activity BGSC runs — football tournaments, esports competitions, fitness challenges. Events have a type: `DE` (Direct Event, no leaderboard) or `LE` (Leaderboard Event).

#### Data model additions

```prisma
model Event {
  id                    String      @id @default(cuid())
  title                 String
  description           String
  type                  EventType   // LE | DE | ALL | DLL
  status                EventStatus @default(UPCOMING)
  startDate             DateTime
  endDate               DateTime
  registrationDeadline  DateTime
  venue                 String?
  rulesPdfUrl           String?
  isTeamed              Boolean     @default(false)
  maxParticipants       Int?
  createdBy             String      // user ID
  createdAt             DateTime    @default(now())

  registrations         Registration[]
}

model Registration {
  id          String   @id @default(cuid())
  userId      String
  eventId     String
  role        String?  // "captain" | "member" (for team events in Phase 3)
  registeredAt DateTime @default(now())

  user        User     @relation(...)
  event       Event    @relation(...)
  @@unique([userId, eventId])  // one registration per user per event
}

enum EventType   { LE DE ALL DLL }
enum EventStatus { UPCOMING ONGOING PAST CANCELLED }
```

#### Endpoints to build

| Endpoint | Auth | What it does |
|----------|------|-------------|
| `POST /events` | Core+ | Create an event |
| `GET /events` | Public | List events; query params: `?status=upcoming&type=LE` |
| `GET /events/:id` | Public | Get one event's full details |
| `PATCH /events/:id` | Core+ (assigned to event) | Update event details |
| `POST /events/:id/register` | User | Register for event (solo only in MVP) |
| `DELETE /events/:id/register` | User | Cancel registration |
| `GET /events/:id/participants` | Public | List registered participants |
| `POST /events/:id/complete` | Core+ | Mark event as complete, trigger fan awards |
| `POST /events/:id/scores` | Core+ | For LE events: manually enter scores |
| `GET /events/:id/leaderboard` | Public | For LE events: ranked participants |

**Registration flow with event bus:**

```typescript
// event.service.ts
async registerForEvent(userId: string, eventId: string) {
  const event = await this.prisma.event.findUniqueOrThrow({ where: { id: eventId } });

  // 1. Validate
  if (event.status !== 'UPCOMING') throw new BadRequestException('Registration is closed.');
  if (new Date() > event.registrationDeadline) throw new BadRequestException('Deadline passed.');

  // 2. Check duplicate
  const existing = await this.prisma.registration.findUnique({
    where: { userId_eventId: { userId, eventId } }
  });
  if (existing) throw new ConflictException('Already registered.');

  // 3. Create registration
  const registration = await this.prisma.registration.create({
    data: { userId, eventId }
  });

  // 4. Publish event (Points service will hear this and award points)
  this.eventBus.publish('RegistrationCreated', { userId, eventId, registrationId: registration.id });

  return registration;
}
```

**✅ Done when:**
- `GET /events?status=upcoming` returns a list (works without login)
- `POST /events/:id/register` requires JWT and creates a registration
- Registering twice returns 409 Conflict
- After `POST /events/:id/complete`, the Points service awards points (check the user's balance)

---

### Milestone 1.3 — Points Service 🔴 MUST

> **What are Points?** A campus currency. Users earn them by participating in events and completing challenges. They spend them in the Store and invest them in leaderboards.

#### Data model additions

```prisma
model PointTransaction {
  id          String   @id @default(cuid())
  userId      String
  amount      Int      // positive = earned, negative = spent
  type        TransactionType
  source      String   // "event_registration" | "challenge" | "admin_award" | "store_redemption"
  referenceId String?  // event ID or challenge ID this relates to
  note        String?
  createdAt   DateTime @default(now())
  user        User     @relation(...)
}

enum TransactionType { EARN SPEND REFUND }
```

#### Points service logic

```typescript
// points.service.ts
@Injectable()
export class PointsService {
  constructor(
    private prisma: PrismaClient,
    private eventBus: EventBusService,
  ) {
    // Subscribe to relevant events
    this.eventBus.subscribe('RegistrationCreated', this.onRegistration.bind(this));
    this.eventBus.subscribe('EventCompleted', this.onEventCompleted.bind(this));
  }

  private async onRegistration({ userId, eventId }: any) {
    await this.award(userId, 10, 'event_registration', eventId, 'Registered for event');
  }

  private async onEventCompleted({ winnerId, eventId }: any) {
    if (winnerId) {
      await this.award(winnerId, 100, 'event_win', eventId, 'Event winner bonus');
    }
  }

  async award(userId: string, amount: number, source: string, referenceId: string, note: string) {
    // Use a transaction so balance is always consistent
    return this.prisma.$transaction([
      this.prisma.pointTransaction.create({
        data: { userId, amount, type: 'EARN', source, referenceId, note }
      }),
      this.prisma.user.update({
        where: { id: userId },
        data: { pointsBalance: { increment: amount } }
      })
    ]);
  }
}
```

#### Endpoints

| Endpoint | Auth | What it does |
|----------|------|-------------|
| `GET /points/balance` | User | My current points balance |
| `GET /points/transactions` | User | My transaction history (paginated) |
| `POST /points/award` | Coordinator+ | Admin manually awards points to any user |

---

### Milestone 1.4 — User Profile & Player Card 🟡 SHOULD

> **What is a Player Card?** A shareable digital card showing a user's avatar, username, sponsor badge, and stats. Think of a trading card for campus athletes.

#### Additional endpoints to add to User Service

| Endpoint | Auth | What it does |
|----------|------|-------------|
| `GET /users/:id/card` | Public | Returns JSON for rendering the player card |
| `PATCH /users/me/interests` | User | Update interests array |
| `GET /users/me/sponsor-stats` | User | My fan count and sponsor ranking |
| `GET /interests` | Public | List all available interest options |

**Player card response shape:**

```typescript
interface PlayerCardResponse {
  username: string;
  avatarUrl: string;
  bio: string;
  interests: string[];        // ["football", "valorant"]
  customTags: string[];       // ["Striker", "IGL"]
  socialLinks: {
    instagram?: string;
    discord?: string;
    twitch?: string;
  };
  sponsor: {
    name: string;
    logoUrl: string;
    fanCount: number;         // this user's contribution
    sponsorRank: number;      // sponsor's current leaderboard rank
  } | null;
  stats: {
    eventsParticipated: number;
    eventsWon: number;
    pointsBalance: number;
    challengesCompleted: number;
  };
}
```

---

### Milestone 1.5 — Announcements & Notifications 🟡 SHOULD

> **What are Announcements?** Official messages from coordinators broadcast to all users (like a college notice board). For MVP, no WhatsApp integration — coordinators post here and copy-paste to WhatsApp manually.

#### Data model

```prisma
model Announcement {
  id          String   @id @default(cuid())
  title       String
  body        String
  type        String[] // ["BGEC", "FitSoc", "Offside", ...]
  createdBy   String   // user ID of coordinator
  expiresAt   DateTime // 4 months from creation
  createdAt   DateTime @default(now())
}

model Notification {
  id          String   @id @default(cuid())
  userId      String
  type        String
  title       String
  body        String
  isRead      Boolean  @default(false)
  createdAt   DateTime @default(now())
  user        User     @relation(...)
}
```

#### Endpoints

| Endpoint | Auth | What it does |
|----------|------|-------------|
| `POST /announcements` | Coordinator+ | Create announcement |
| `GET /announcements` | Public | List announcements (last 4 months), supports `?type=BGEC` filter |
| `GET /notifications` | User | My unread notifications (count + list) |
| `PATCH /notifications/:id/read` | User | Mark one as read |
| `POST /notifications/read-all` | User | Mark all as read |

**Notification service (subscribes to EventBus):**

```typescript
// notification.service.ts
constructor(private eventBus: EventBusService, private prisma: PrismaClient) {
  this.eventBus.subscribe('RegistrationCreated', this.onRegistration.bind(this));
}

private async onRegistration({ userId, eventId }: any) {
  const event = await this.prisma.event.findUnique({ where: { id: eventId } });
  await this.prisma.notification.create({
    data: {
      userId,
      type: 'EVENT_REGISTRATION',
      title: 'Registration Confirmed! 🎉',
      body: `You're registered for ${event.title}.`,
    }
  });
}
```

---

### Milestone 1.6 — Hall of Fame v1 & Admin Panel 🟡 SHOULD

> **What is the Hall of Fame?** An archive of all event winners and top sponsor contributors. For MVP, winners are manually seeded by admins.

#### Endpoints

| Endpoint | Auth | What it does |
|----------|------|-------------|
| `GET /hall-of-fame/event-winners` | Public | List of past event winners |
| `GET /hall-of-fame/sponsor-champions` | Public | Top contributors per sponsor |
| `POST /hall-of-fame/entries` | Coordinator+ | Manually add a winner entry |
| `GET /admin/users` | Coordinator+ | Paginated user list with filters |
| `PATCH /admin/users/:id/role` | Coordinator+ | Change user role |
| `PATCH /admin/users/:id/status` | Coordinator+ | Suspend/reactivate user |

---

### 🏁 Phase 1 Backend: Acceptance Checklist (Must pass by June 26)

**Test all of these manually in Postman before declaring done:**

```
Authentication
✅ POST /auth/register → creates user, returns tokens
✅ POST /auth/login → returns tokens
✅ POST /auth/login (wrong password) → 401
✅ POST /auth/refresh → rotates refresh token
✅ POST /auth/google → creates or logs in user

Sponsors
✅ GET /sponsors/active → list of active sponsors
✅ POST /users/me/sponsor → affiliates user with sponsor
✅ POST /users/me/sponsor (second call same semester) → 400

Events
✅ GET /events?status=upcoming → public, no auth needed
✅ POST /events (as coordinator) → creates event
✅ POST /events/:id/register (as user) → creates registration
✅ POST /events/:id/register (duplicate) → 409
✅ After registration → user gets +10 points (check GET /points/balance)
✅ After registration → user gets an in-app notification (check GET /notifications)

Profile
✅ GET /users/:id → returns public profile
✅ PATCH /users/me → updates bio/interests
✅ GET /users/:id/card → returns player card JSON

Announcements
✅ POST /announcements (as coordinator) → creates announcement
✅ GET /announcements → public list, no auth needed

Admin
✅ GET /admin/users (as coordinator) → paginated list
✅ GET /admin/users (as regular user) → 403

Deployment
✅ All services deployed to Railway staging
✅ Swagger UI accessible at /api/docs
✅ GitHub Actions CI is green
```

---

## 📱 Phase 1: Frontend MVP (June 27–July 10)

> **Note for frontend devs:** The backend is frozen on June 26. You work against stable APIs. If you find a bug, create a GitHub issue with label `bug` + `backend` — don't ask the backend team to add new endpoints.

### MVVM pattern quick-start

Every screen follows this structure:

```
ScreenName/
├── ScreenName.tsx          ← View (just renders, no logic)
├── ScreenNameViewModel.ts  ← ViewModel (state + actions)
└── screenName.types.ts     ← TypeScript types for this screen
```

**Example: Event list screen**

```typescript
// EventsViewModel.ts
export function useEventsViewModel() {
  const { data: events, isLoading, error } = useQuery({
    queryKey: ['events', 'upcoming'],
    queryFn: () => EventRepository.getEvents({ status: 'upcoming' }),
  });

  const registerForEvent = useMutation({
    mutationFn: (eventId: string) => EventRepository.register(eventId),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['events'] });
      Alert.alert('Success!', 'You are registered.');
    },
  });

  return { events, isLoading, error, registerForEvent };
}

// Events.tsx (View — no API calls here, only ViewModel)
export function EventsScreen() {
  const { events, isLoading, registerForEvent } = useEventsViewModel();

  if (isLoading) return <LoadingSpinner />;

  return (
    <FlatList
      data={events}
      renderItem={({ item }) => (
        <EventCard
          event={item}
          onRegister={() => registerForEvent.mutate(item.id)}
        />
      )}
    />
  );
}
```

### Milestone 1.7 — Mobile App Screens 🔴 MUST

| Screen | What to build |
|--------|--------------|
| **Login / Register** | Two-tab form. Google OAuth button. Sponsor selection on register (dropdown of `GET /sponsors/active`). |
| **Onboarding** | After first login: Interest selection grid → Sponsor selection → (skip friends for MVP) |
| **Home — Landing** | Static intro text + coordinator highlights (use `GET /announcements` for the latest 3) |
| **Home — Announcements** | List with category filter chips. Show relative time ("2 hours ago"). |
| **Events — Browse** | Tab bar: Upcoming / Ongoing / Past. Each tab shows event cards. |
| **Events — Details** | Event info + Register button. If already registered, show "Registered ✓". |
| **User Profile** | Player card (display the JSON from `GET /users/:id/card`). Below: bio, interests. |
| **Edit Profile** | Form to update bio, interests, social links. |
| **Points page** | Balance number + transaction history list. |
| **Hall of Fame** | Two tabs: Event Winners / Sponsor Champions. |
| **Notifications** | Bell icon badge. Slide-to-dismiss or tap to mark read. |

### Milestone 1.8 — Web Admin Console 🟡 SHOULD

| Screen | What to build |
|--------|--------------|
| **Login** | Simple form. Coordinators/Founders only — redirect back if role < COORDINATOR. |
| **Dashboard** | Stats cards: total users, active events, total registrations. |
| **Events management** | Table of all events. "Create Event" button → form modal. |
| **Announcement creator** | Popup form with rich text (use `react-quill`) and category multi-select. |
| **Users table** | Paginated table with search, role filter, status filter. |
| **Sponsor management** | Create/edit sponsors, set tenure dates, view leaderboard. |

---

## 👥 Phase 2: Community & Engagement (July 11–August 21)

Phase 2 focuses on retention — giving users reasons to come back daily.

### What's new in Phase 2

| Feature | Why it matters |
|---------|---------------|
| **Friends system** | Social graph — find teammates, see what others are playing |
| **Social feed v2** | Create posts with images, likes, comments |
| **Challenges** | Short tasks users can complete for bonus points |
| **Push notifications (FCM)** | Notify users even when app is closed |
| **Kafka event bus** | Replace the in-memory EventEmitter with real, durable events |

### Kafka migration (beginner guide)

> Kafka is a real message broker. Unlike the in-memory EventEmitter, it survives server restarts and can be consumed by multiple services simultaneously.

```bash
# Add Kafka to docker-compose.yml
services:
  kafka:
    image: bitnami/kafka:latest
    environment:
      - KAFKA_CFG_ZOOKEEPER_CONNECT=zookeeper:2181
    ports:
      - "9092:9092"
  zookeeper:
    image: bitnami/zookeeper:latest
    environment:
      - ALLOW_ANONYMOUS_LOGIN=yes
```

```typescript
// Replace EventBusService with KafkaEventBus
import { Kafka } from 'kafkajs';

@Injectable()
export class KafkaEventBus {
  private kafka = new Kafka({ brokers: [process.env.KAFKA_BROKER] });
  private producer = this.kafka.producer();

  async publish(topic: string, payload: object) {
    await this.producer.connect();
    await this.producer.send({
      topic,
      messages: [{ value: JSON.stringify(payload) }],
    });
  }
}
```

> **💡 Beginner tip:** You don't need to understand all of Kafka's internals. The team will provide a `publish()` and `subscribe()` wrapper function. Use those and don't worry about producers/consumers/offsets until you're curious.

### Milestone 2.1 — Friends System

| Endpoint | Auth | What it does |
|----------|------|-------------|
| `POST /friends/request/:userId` | User | Send friend request |
| `POST /friends/accept/:requestId` | User | Accept incoming request |
| `DELETE /friends/:requestId` | User | Reject / cancel request |
| `GET /friends` | User | My friends list |
| `GET /friends/suggestions` | User | Suggested friends based on shared interests + sponsor |
| `GET /friends/requests` | User | Incoming friend requests |

### Milestone 2.2 — Social Feed

> **Visibility levels:** `PUBLIC` (everyone including guests), `PROTECTED` (logged-in users only), `PRIVATE` (only your friends).

| Endpoint | Auth | What it does |
|----------|------|-------------|
| `POST /posts` | User | Create a post (image + caption + visibility) |
| `GET /feed` | User | Aggregated feed: friends' posts + public posts |
| `POST /posts/:id/like` | User | Like a post |
| `DELETE /posts/:id/like` | User | Unlike a post |
| `POST /posts/:id/comments` | User | Add a comment |
| `GET /posts/:id/comments` | Public/User | Get comments (respects visibility) |

### Milestone 2.3 — Challenges

| Endpoint | Auth | What it does |
|----------|------|-------------|
| `GET /challenges` | Public | Browse challenges (filter by domain, difficulty) |
| `POST /challenges/:id/accept` | User | Accept a challenge |
| `POST /challenges/:id/submit` | User | Submit completion proof |
| `GET /challenges/submissions` | User | My submission history |
| `PATCH /challenges/submissions/:id` | Core+ | Approve or reject submission |

---

## ⚙️ Phase 3: Operations & League Management (August 22–October 16)

This phase is primarily for the **organizing committee crew**. After Phase 3, the Union Page, auctions, and full league management are operational.

### What's new in Phase 3

| Feature | Plain English |
|---------|--------------|
| **Union Page** | A Notion/Jira hybrid for the crew — tasks, Kanban, Gantt charts |
| **Team formation** | Team creation, invite codes, captain management |
| **Bracket generator** | Auto-generates tournament brackets (Round Robin, Single/Double Elimination) |
| **Auction system** | Live auction where captains bid for players using a purse |
| **Elasticsearch** | Upgraded search — fuzzy matching, typo tolerance |

### Auction system (beginner overview)

The auction is the most complex feature. Here's how it works:

```
1. Admin sets up auction: defines captains, player base prices, purse (= K × sum of base prices)
2. Admin starts auction via web console
3. A player appears on the "block"
4. Captains see the player in real-time (WebSocket) and place bids
5. Server-authoritative 5-second countdown resets on each bid
6. When countdown hits 0, player is "sold" to highest bidder
7. Captain's purse decreases by the bid amount
8. WebSocket broadcasts the result to all viewers
```

**WebSocket rooms (Socket.io):**

```typescript
// Server
@WebSocketGateway({ namespace: '/auction' })
export class AuctionGateway {
  @WebSocketServer() server: Server;

  @SubscribeMessage('join_auction')
  handleJoin(@ConnectedSocket() client: Socket, @MessageBody() auctionId: string) {
    client.join(`auction:${auctionId}`);
  }

  broadcastBid(auctionId: string, bid: any) {
    this.server.to(`auction:${auctionId}`).emit('bid_placed', bid);
  }
}

// Client (React Native)
const socket = io('https://api.bgsc.com/auction');
socket.emit('join_auction', auctionId);
socket.on('bid_placed', (bid) => setBids(prev => [bid, ...prev]));
```

---

## 🔌 Phase 4: Integrations & Polish (October 17–November 13)

| Integration | How it works |
|-------------|-------------|
| **Strava** | User connects their Strava account (OAuth). We pull their recent activities daily and display on their profile. |
| **Steam** | User connects Steam (OpenID). We show their owned games and playtime. |
| **Google Calendar** | Two-way sync — task deadlines appear in the crew's Google Calendar. |
| **WhatsApp Business API** | When a coordinator posts an announcement tagged "BGEC", it auto-sends to the BGEC WhatsApp community group. |
| **Full media pipeline** | Images processed (resized to 3 sizes, WebP-converted), stored on S3, served via CDN. |

---

## 🧪 Phase 5: Buffer & Contingency (November 14–December 11)

**No new features are built in this phase.** Use this time to:

| Activity | Who | Duration |
|----------|-----|----------|
| Bug bash — every team member tests the full app | All devs | 1 week |
| Load testing with k6 (target: 1000 concurrent users) | Backend team | 3 days |
| Finalise API docs (Swagger), user guides, admin runbooks | All | 1 week |
| Soft launch with 100 beta users (campus ambassadors) | Product + marketing | 1 week |
| App store submission | Mobile lead | 3 days |

**Load test script (k6):**

```javascript
// k6-load-test.js
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  vus: 1000,          // 1000 virtual users
  duration: '60s',
};

export default function () {
  const response = http.get('https://api.bgsc.com/events?status=upcoming');
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  sleep(1);
}
```

---

## 🛠️ Developer Onboarding (Step by Step)

### Day 1: Get running

```bash
# 1. Clone
git clone https://github.com/bgsc/bgsc-platform.git
cd bgsc-platform

# 2. Install all dependencies
cd backend && npm install
cd ../mobile && npm install
cd ../web && npm install

# 3. Copy environment template
cp backend/.env.example backend/.env
# Edit .env with values from the team's shared secrets doc

# 4. Start all services
cd backend
docker-compose up -d
# Wait ~30 seconds for Postgres to start, then:
npx prisma migrate dev

# 5. Verify everything works
curl http://localhost:3000/api/health
# Should return: {"status":"ok"}
```

### Day 1: Verify Swagger is working

Open in browser: `http://localhost:3000/api/docs`

You should see an interactive API explorer. Use it to test endpoints manually.

### Code quality commands

```bash
# Format code before committing
npm run format

# Run linter
npm run lint

# Run tests
npm test

# Run tests with coverage report
npm run test:cov
```

### Git workflow

```bash
# Always branch from dev
git checkout dev
git pull origin dev
git checkout -b feature/1234-event-registration

# Commit often (small, descriptive commits)
git add .
git commit -m "feat: add registration deadline validation"

# Push and open a PR to dev
git push origin feature/1234-event-registration
# → Open PR on GitHub → dev
```

**Branch naming:**
- `feature/XXX-short-description` — new feature
- `fix/XXX-bug-description` — bug fix
- `chore/XXX-task` — maintenance (deps, config, docs)

---

## ❓ FAQ for Beginners

**Q: I've never built a microservices app. Where do I start?**

A: Start with the Auth Service — it's self-contained and well-documented. Once you understand how a NestJS module, controller, and service work together there, the pattern repeats everywhere. The NestJS docs have a great [first steps guide](https://docs.nestjs.com/first-steps).

---

**Q: What's Prisma and why not raw SQL?**

A: Prisma is an ORM (Object-Relational Mapper). It lets you write TypeScript code instead of SQL. For example:

```typescript
// Without Prisma (raw SQL)
const result = await db.query('SELECT * FROM users WHERE id = $1', [userId]);

// With Prisma
const user = await prisma.user.findUnique({ where: { id: userId } });
```

The Prisma version has type safety — TypeScript knows exactly what fields `user` has.

---

**Q: I don't understand JWT. What is it?**

A: Imagine a signed wristband at a concert. The wristband proves you're allowed to be there. When you log in, the server gives you a JWT (the wristband). On every request after that, you send the JWT in the `Authorization` header. The server reads it to know who you are — no database lookup needed.

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

The JWT expires in 15 minutes (to limit damage if stolen). You use the `refresh token` (lasts 7 days) to get a new one.

---

**Q: I've never used Kafka. How do I start?**

A: In Phase 1, you won't touch Kafka at all. We use Node's built-in `EventEmitter`. Kafka only comes in Phase 2. When it does, the team will provide a wrapper service (`EventBusService`) with `publish()` and `subscribe()` methods. You just call those — no Kafka internals needed.

---

**Q: What if I don't know Kubernetes?**

A: You don't need it for MVP. Everything deploys to Railway (one-click). Kubernetes is a Phase 4+ concern handled by the DevOps lead.

---

**Q: How do I test event-driven flows locally?**

A: The in-memory event bus is synchronous in tests:

```typescript
// In your test
it('should award points when user registers', async () => {
  // Act
  await eventService.registerForEvent(userId, eventId);

  // Assert (EventEmitter is synchronous, so points are already awarded)
  const user = await prisma.user.findUnique({ where: { id: userId } });
  expect(user.pointsBalance).toBe(10);
});
```

---

**Q: I found a bug in someone else's code. What do I do?**

A: Create a GitHub Issue with:
- Title: `[Bug] Short description`
- Labels: `bug`, `backend` or `frontend`, phase label
- Description: Steps to reproduce, expected behavior, actual behavior, any error messages

Don't fix other people's bugs without telling them first (they may already be working on it).

---

**Q: Where do I log things so they show up in monitoring?**

A: Use NestJS's built-in logger:

```typescript
import { Logger } from '@nestjs/common';

@Injectable()
export class EventService {
  private logger = new Logger(EventService.name);

  async registerForEvent(userId: string, eventId: string) {
    this.logger.log(`User ${userId} registering for event ${eventId}`);
    // ...
    this.logger.error(`Registration failed: ${error.message}`, error.stack);
  }
}
```

Errors automatically appear in Sentry. Logs appear in Railway's log viewer.

---

## 📚 Learning Resources

### Backend

| Topic | Resource | Time to complete |
|-------|---------|----------------|
| NestJS full course | [Official docs](https://docs.nestjs.com) (read: Overview, Controllers, Providers, Modules) | 4 hours |
| Prisma tutorial | [Prisma Getting Started](https://www.prisma.io/docs/getting-started) | 2 hours |
| JWT deep dive | [jwt.io](https://jwt.io/introduction) | 1 hour |
| Redis crash course | [Redis Quickstart](https://redis.io/docs/getting-started/) | 1 hour |
| REST API design | [RESTful API Design](https://restfulapi.net) | 1 hour |

### Frontend (Mobile)

| Topic | Resource | Time to complete |
|-------|---------|----------------|
| React Native fundamentals | [Expo tutorial](https://docs.expo.dev/tutorial/introduction/) | 3 hours |
| React Query | [TanStack Query docs](https://tanstack.com/query/latest/docs/react/overview) | 2 hours |
| Zustand | [Zustand docs](https://zustand-demo.pmnd.rs) | 1 hour |
| React Navigation | [React Navigation docs](https://reactnavigation.org/docs/getting-started) | 2 hours |

### DevOps / General

| Topic | Resource | Time to complete |
|-------|---------|----------------|
| Docker in 100 seconds | [Fireship Docker video](https://youtu.be/Gjnup-PuquQ) | 2 min |
| Git branching | [Learn Git Branching (interactive)](https://learngitbranching.js.org) | 1 hour |
| GitHub Actions | [GitHub Actions quickstart](https://docs.github.com/en/actions/quickstart) | 1 hour |

---

## 🎯 Final Checklist Before Each Phase Release

```
Code Quality
✅ All tests pass: npm test (no failures, coverage > 50%)
✅ Linter clean: npm run lint (zero errors)
✅ No hardcoded secrets (npm run scan:secrets)

API
✅ All new endpoints documented in Swagger (/api/docs)
✅ Postman collection updated and shared with team
✅ All endpoints return correct HTTP status codes
✅ Error messages are user-friendly (not stack traces)

Database
✅ All migrations run cleanly: npx prisma migrate deploy
✅ Migrations are idempotent (running twice doesn't fail)
✅ Seed data updated if new required data added

Deployment
✅ Deployed to Railway staging (not just local)
✅ Health check endpoint returns 200: GET /api/health
✅ Sentry reports no new errors after deploy

Security
✅ No endpoint is accidentally public that should be protected
✅ JWT expiry is 15 minutes (not longer)
✅ Refresh tokens are rotated on each use
```

---

*Built with ❤️ by BGSC Dev Team — BITS Goa*

*If you're stuck: open a GitHub Issue, post in `#dev-help`, or reach out to the lead for your service. No question is too basic.*

**Let's build something legendary. 🚀**
