# FeedbackPro Development Breakdown: V1.0 (Monorepo - Separate Backend Architecture)

---

**Detailed Development Breakdown: FeedbackPro V1.0 (Monorepo - Separate Backend Architecture - Refined)**

**(As of: Sunday, April 20, 2025 at 11:13:30 PM +05 - Jizzakh, Jizzakh Region, Uzbekistan)**

**1. Introduction & Guiding Principles**

This document outlines the detailed phases and tasks for developing FeedbackPro V1.0 using a decoupled architecture managed within a **Turborepo monorepo**. It serves as a specific guide for implementation based on the following principles and technical decisions:

- **Architecture:** Separate Frontend (Next.js v15.1 App Router) and Backend (Node.js + Express.js). Communication via REST API + JWT.
- **Repository Structure:** **Monorepo** managed with **Turborepo** and **pnpm/yarn/npm workspaces**. Code organized into `apps/frontend`, `apps/backend`, and shared `packages/*`.
- **Project Structure (FE):** Standard Next.js App Router structure **without** a `src` directory within `apps/frontend`.
- **Project Structure (BE):** Standard Node.js structure, likely using a `src` directory within `apps/backend`.
- **Database:** PostgreSQL with Prisma ORM (managed by the backend app).
- **Authentication:** JWT-based. Backend (Node.js/Passport.js) issues tokens (**via secure HttpOnly cookies**). Frontend relies on browser cookie handling for sending tokens. Backend middleware verifies tokens. Frontend client state manages user info/auth status.
- **API Layer:** RESTful API exposed by the Node.js/Express backend (`/api/v1/...`).
- **Data Fetching Strategy (FE):**
  - **Server Components:** Use direct server-side data fetching (calling server functions) ONLY for _publicly accessible data_ or data needed before hydration where auth is not a factor. For authenticated initial data, prefer client-side fetching (see below).
  - **Client Components:** Use **TanStack Query (React Query V5)** to call _external_ backend API routes for **all authenticated data fetching** and all mutations, managing client-side state.
- **Styling:** Tailwind CSS v3 with Shadcn UI (v2.3.0 or compatible) within the frontend app.
- **Component Model:** Next.js Server/Client component best practices. Route components (`page.js`, `layout.js`) remain Server Components where possible.
- **Routing & Layouts:** Leverage Next.js Route Groups and nested `layout.js` files within the frontend app.
- **State Management (FE):** TanStack Query (server state), Zustand/Context (auth status/user info), React `useState` (local state).
- **Forms & Validation:** React Hook Form + Zod. **Zod schemas defined in shared `packages/validators`**.
- **Loading/Error Handling:** Real-time async loading (spinner) & error feedback (toasts) via TanStack Query in FE Client Components. Page loaders (`loading.js`) deferred.
- **Development Practices:** Consistency, Modularity, Reusability (leveraging shared packages). Just-in-Time package installs/folder creation. Use `turbo` commands.
- **Deferred (V1.0):** Automated Testing, API Documentation (Swagger/OpenAPI).

**Prerequisites (Assumed Done Before Phase 1):**

- Git initialized in the root monorepo directory.
- Node.js and pnpm (or yarn/npm) installed.
- Next.js 15.1 frontend code (App Router, no `src`, alias) placed within `apps/frontend`.
- Tailwind CSS v3 integrated within `apps/frontend`.
- Shadcn UI v2.3.0 initialized within `apps/frontend`.

**Assumptions for AI Agent:**

- Understands monorepo concepts, Turborepo, pnpm workspaces.
- Understands project context, separate FE/BE architecture, specific tech stack.
- Follows best practices, modularity, uses env vars. Uses `'use client'` where needed. Creates directories implicitly.
- Uses appropriate `pnpm --filter <app|package>` commands for installs.

---

**Phase 1: Monorepo Setup, Backend Foundation & Core Providers**

_Goal: Initialize Turborepo, set up backend basics (Express, Prisma, CORS), configure core FE providers, manage initial environment variables._

- **Task 1.1 (Monorepo):** Set up pnpm workspaces (`pnpm-workspace.yaml`, root `package.json`). Install Turborepo: `pnpm add -w -D turbo`.
- **Task 1.2 (Monorepo):** Initialize Turborepo configuration (`turbo.json`) defining basic pipelines (`dev`, `build`, `lint`).
- **Task 1.3 (BE):** Create `apps/backend`. Initialize Node.js project (`cd apps/backend && pnpm init`). Set up `src` directory structure.
- **Task 1.4 (BE):** Install core BE dependencies: `pnpm add express cors dotenv pino pino-pretty --filter backend`.
- **Task 1.5 (BE):** Install Prisma: `pnpm add @prisma/client --filter backend` & `pnpm add -D prisma --filter backend`. Initialize Prisma (`cd apps/backend && npx prisma init`).
- **Task 1.6 (Setup):** Create initial environment variable files: `apps/backend/.env`, `apps/backend/.env.example`, `apps/frontend/.env.local`, `apps/frontend/.env.example`. Add comments for required variables.
- **Task 1.7 (BE):** Configure `DATABASE_URL` in `apps/backend/.env`.
- **Task 1.8 (DB):** Define initial Prisma schemas (`apps/backend/prisma/schema.prisma`) for `User` (with `hashedPassword`) and `Business`.
- **Task 1.9 (DB):** Run initial migration: `cd apps/backend && npx prisma migrate dev --name init`. Generate client: `npx prisma generate`.
- **Task 1.10 (BE):** Create Prisma client singleton (`apps/backend/src/config/db.js`).
- **Task 1.11 (BE):** Create basic Express server entry point (`apps/backend/src/server.js`). Configure middleware: `cors` (**explicitly configure origin based on `process.env.CORS_ORIGIN` for FE dev URL `http://localhost:3000`, allow `credentials: true`**), `express.json()`, etc. Load env vars. Set up logger. Add note about tightening CORS for production later.
- **Task 1.12 (BE):** Set up routing prefix (`/api/v1`). Implement health check route. Add `PORT` to backend `.env`.
- **Task 1.13 (FE):** Install FE dependencies: `pnpm add zustand @tanstack/react-query lucide-react --filter frontend`.
- **Task 1.14 (FE):** Add `NEXT_PUBLIC_API_BASE_URL=http://localhost:[BACKEND_PORT]/api/v1` (replace port) to `apps/frontend/.env.local`.
- **Task 1.15 (FE):** Create root layout (`app/layout.js`) _(Server Component)_.
- **Task 1.16 (FE):** Create `Providers` component (`components/providers/Providers.jsx`) _(Client Component - 'use client')_. Include `QueryClientProvider`, `Toaster`.
- **Task 1.17 (FE):** Use `Providers` in root layout wrapping `{children}`.
- **Task 1.18 (FE):** Create reusable `LoadingSpinner` component (`components/ui/LoadingSpinner.jsx`).
- **Task 1.19 (FE):** Create `(marketing)` route group & layout (`app/(marketing)/layout.js`) _(Server Component)_.
- **Task 1.20 (FE):** Create marketing home page: (`app/(marketing)/page.js`) _(Server Component)_.
- **Task 1.21 (Monorepo):** Configure root `dev` script in root `package.json` (`"dev": "turbo dev"`) and define the `dev` pipeline in `turbo.json` to run FE/BE dev servers concurrently.

- **End of Phase Check:** `pnpm dev` starts both. FE homepage visible. BE health check responds. Basic CORS configured. Prisma connects. Env vars setup.

---

**Phase 2: Authentication (Business Owner & Admin)**

_Goal: Implement JWT-based registration/login flow using Passport.js on BE, manage client auth state via Zustand/Context._

- **Task 2.1 (BE):** Install BE auth dependencies: `pnpm add passport passport-jwt jsonwebtoken bcrypt zod --filter backend`.
- **Task 2.2 (DB):** Update `User` schema (`role`, `isActive`). Migrate & generate.
- **Task 2.3 (BE):** Implement `bcrypt` hashing utility (`apps/backend/src/utils/hashPassword.js`).
- **Task 2.4 (BE):** Configure Passport JWT strategy (`apps/backend/src/config/passport.js`) to verify tokens using `JWT_SECRET` from `.env`.
- **Task 2.5 (BE/Service):** Create `authService` (`apps/backend/src/services/authService.js`) for register/login logic (hash, compare, issue JWT including user ID/role).
- **Task 2.6 (BE/Controller):** Create `authController` (`apps/backend/src/controllers/authController.js`). Handle requests, validate with Zod (from shared package), call service, **set HttpOnly, Secure, SameSite=Lax cookie for access token on successful login response**. Return user info (excluding sensitive data) in login response body.
- **Task 2.7 (Package):** Create shared validators package (`packages/validators`). Define Zod schemas for Auth. Install `zod`. Import schema into BE controller.
- **Task 2.8 (BE/Routes):** Create Auth routes (`apps/backend/src/routes/authRoutes.js`) for POST `/api/v1/auth/register` and POST `/api/v1/auth/login`.
- **Task 2.9 (BE):** Create JWT verification middleware (`apps/backend/src/middleware/authMiddleware.js`) using `passport.authenticate('jwt', { session: false })`.
- **Task 2.10 (FE):** Install form libraries: `pnpm add react-hook-form @hookform/resolvers --filter frontend`. Install shared validator: `pnpm add @repo/validators --filter frontend`.
- **Task 2.11 (FE):** Create API client functions (`apps/frontend/lib/apiClient/auth.js`) for register/login. **Ensure `credentials: 'include'` is set in fetch options** to send/receive cookies.
- **Task 2.12 (FE):** Set up client-side auth state management (`apps/frontend/store/authStore.js` using Zustand or Context). Store `isAuthenticated`, `user` (basic info like id, email, role).
- **Task 2.13 (FE):** Create `(auth)` route group & layout (`apps/frontend/app/(auth)/layout.js`) _(Server Component)_.
- **Task 2.14 (FE):** Implement `RegisterPage` (`apps/frontend/app/(auth)/register/page.js`) _(Server Component)_ rendering `RegisterForm`.
- **Task 2.15 (FE):** Create `RegisterForm` component (`apps/frontend/components/auth/RegisterForm.jsx`) _(Client Component - 'use client')_. Uses Shadcn, RHF+Zod (import from `@repo/validators`). **Uses `useMutation` (TanStack) calling `apiClient.register`**. Shows `LoadingSpinner`, uses `toast`.
- **Task 2.16 (FE):** Implement `LoginPage` (`apps/frontend/app/(auth)/login/page.js`) _(Server Component)_ rendering `LoginForm`.
- **Task 2.17 (FE):** Create `LoginForm` component (`apps/frontend/components/auth/LoginForm.jsx`) _(Client Component - 'use client')_. Uses Shadcn, RHF+Zod (import from `@repo/validators`). **Uses `useMutation` (TanStack) calling `apiClient.login`**. On success, **update client auth store with user info from response body** and redirect (`useRouter`). Handles loading/errors (`LoadingSpinner`/`toast`).
- **Task 2.18 (FE):** Create `(app)` route group & layout (`apps/frontend/app/(app)/layout.js`) _(Server Component)_. Include placeholders.
- **Task 2.19 (FE):** Implement basic client-side protected route logic (e.g., in a wrapper component within `(app)/layout.js` or using middleware if preferred) checking auth store state. Redirect if not authenticated.
- **Task 2.20 (FE):** Update Header/Navbar _(Client Component)_: Use auth store state. Implement logout action (clear client state, maybe call backend logout later).

- **End of Phase Check:** Registration/Login via BE API works. HttpOnly cookie set by BE. Client state updated based on login response payload. Basic FE protection works.

---

**Phase 3: Core Survey Management (Business Owner)**

_Goal: Implement survey creation (API + TanStack) and listing (Client Component Fetch via TanStack)._

- **Task 3.1 (DB):** Define `Survey` schema. Migrate & generate.
- **Task 3.2 (BE/Service):** Implement `surveyService` (`apps/backend/src/services/surveyService.js`). Check ownership via `req.user.id` from JWT middleware.
- **Task 3.3 (BE/Controller):** Implement `surveyController`. Use Zod schemas from `@repo/validators` if applicable.
- **Task 3.4 (BE/Routes):** Implement authenticated survey routes (`apps/backend/src/routes/surveyRoutes.js`) for POST `/`, GET `/`, GET `/:id`. **Apply JWT auth middleware.**
- **Task 3.5 (FE):** Create `apiSurveys` client functions (`apps/frontend/lib/apiClient/surveys.js`). Use `credentials: 'include'`.
- **Task 3.6 (FE):** Enhance `(app)` layout with `Sidebar`.
- **Task 3.7 (FE):** Implement `Sidebar` component (`apps/frontend/components/layout/Sidebar.jsx`) _(Client)_.
- **Task 3.8 (FE):** Implement `DashboardPage` (`apps/frontend/app/(app)/dashboard/page.js`) _(Server Component)_. This page will render the client component responsible for fetching/displaying surveys.
- **Task 3.9 (FE):** Create `SurveyListLoader` component (`apps/frontend/components/dashboard/SurveyListLoader.jsx`) _(Client Component - 'use client')_.
  - **Uses `useQuery` (TanStack) calling `apiClient.getSurveys`** to fetch the survey list.
  - Handles loading (`LoadingSpinner`) and error states (`toast`).
  - Renders the actual list display component (`SurveyListDisplay`?) passing the fetched data.
- **Task 3.10 (FE):** Create `SurveyListDisplay` component (`apps/frontend/components/dashboard/SurveyListDisplay.jsx`) _(Client or Server Component)_. Receives `surveys` prop. Displays using Shadcn `Table`/`Card`. Includes `Link` to create.
- **Task 3.11 (FE):** Implement `CreateSurveyPage` (`apps/frontend/app/(app)/surveys/create/page.js`) _(Server)_ rendering `CreateSurveyForm`.
- **Task 3.12 (FE):** Create `CreateSurveyForm` component (`apps/frontend/components/surveys/CreateSurveyForm.jsx`) _(Client)_. Uses RHF+Zod (`@repo/validators`). **Uses `useMutation` (TanStack) calling `apiClient.createSurvey`**. **On success, call `queryClient.invalidateQueries(['surveys'])`** (adjust query key as needed). Shows `LoadingSpinner`, uses `toast`. Redirects.
- **Task 3.13 (FE):** Implement `SurveyDetailPage` (`apps/frontend/app/(app)/surveys/[surveyId]/page.js`) _(Server Component)_. Renders a client component loader (`SurveyDetailLoader`) passing the `surveyId`.
- **Task 3.14 (FE):** Create `SurveyDetailLoader` component (`apps/frontend/components/surveys/SurveyDetailLoader.jsx`) _(Client)_. Props: `surveyId`. **Uses `useQuery` (TanStack) calling `apiClient.getSurveyById(surveyId)`**. Handles loading/error. Renders `SurveyDetailsDisplay` passing data.
- **Task 3.15 (FE):** Create `SurveyDetailsDisplay` component (`apps/frontend/components/surveys/SurveyDetailsDisplay.jsx`) _(Client or Server)_. Receives `survey` prop. Renders details, `SmsSendForm`, `SmsSendList`, `QrCodeGenerator`.

- **End of Phase Check:** BO Dashboard and Survey Detail pages now load initial authenticated data using client-side TanStack Query. Creation mutation works and invalidates list query.

---

**Phase 4: Consumer Survey Form**

_Goal: Allow public survey viewing/submission via BE API + TanStack Query._

- **Task 4.1 (DB):** Define `SurveyResponse` schema. Migrate & generate.
- **Task 4.2 (BE/Service):** Implement `responseService` (submit, get public view).
- **Task 4.3 (BE/Controller):** Implement `responseController`.
- **Task 4.4 (BE/Routes):** Implement **public** survey routes (`apps/backend/src/routes/publicSurveyRoutes.js?`) for GET view, POST submit.
- **Task 4.5 (FE):** Create `apiPublicSurveys` client functions.
- **Task 4.6 (FE):** Implement `PublicSurveyPage` (`apps/frontend/app/feedback/[surveyId]/page.js`) _(Server Component)_. Render `SurveyForm`. Use `(feedback)` layout.
- **Task 4.7 (FE):** Create `SurveyForm` component (`apps/frontend/components/feedback/SurveyForm.jsx`) _(Client)_. **Uses `useQuery` (TanStack) calling `apiClient.getSurveyStructure`**. Renders form. **Uses `useMutation` (TanStack) calling `apiClient.submitResponse`**. Shows success state/toast. Shows `LoadingSpinner`, uses `toast`.

- **End of Phase Check:** Public survey form functional via TanStack Query.

---

**Phase 5: QR Code Feedback Collection**

_Goal: Generate QR codes via authenticated BE API + TanStack Query._

- **Task 5.1 (BE):** Install `qrcode`: `pnpm add qrcode --filter backend`.
- **Task 5.2 (BE/Controller):** Implement QR code logic in `surveyController`.
- **Task 5.3 (BE/Routes):** Implement authenticated GET `/api/v1/surveys/:surveyId/qrcode`. **Apply JWT auth middleware**.
- **Task 5.4 (BE/Service):** Ensure `responseService` sets method correctly.
- **Task 5.5 (FE):** Add `getSurveyQrCode` to `apiSurveys` client.
- **Task 5.6 (FE):** Create `QrCodeGenerator` component (`apps/frontend/components/surveys/QrCodeGenerator.jsx`) _(Client)_. Render on `SurveyDetailsDisplay`. **Uses `useQuery` (TanStack, on click) calling `apiClient.getSurveyQrCode`**. Displays QR. Shows loading, uses `toast`.

- **End of Phase Check:** QR codes generated via authenticated BE route.

---

**Phase 6: SMS Feedback Collection & Tracking**

_Goal: Send SMS via BE API + TanStack, handle SMS links, track status via BE API + TanStack._

- **Task 6.1 (BE):** Install SMS SDK. Implement/Configure `smsService`. Use env vars.
- **Task 6.2 (DB):** Define `SmsSend` schema. Migrate & generate.
- **Task 6.3 (BE/Service):** Implement SMS sending logic.
- **Task 6.4 (BE/Controller):** Implement SMS send controller action.
- **Task 6.5 (BE/Routes):** Implement authenticated POST `/api/v1/surveys/:surveyId/send-sms`. **Apply JWT auth middleware**.
- **Task 6.6 (BE/API):** Implement public GET `/api/v1/public/feedback/sms/[identifier]`.
- **Task 6.7 (BE/Service):** Modify `responseService` for SMS submissions.
- **Task 6.8 (BE/Service):** Implement `getSmsSendsBySurvey`.
- **Task 6.9 (BE/Controller):** Implement controller for getting SMS sends.
- **Task 6.10 (BE/Routes):** Implement authenticated GET `/api/v1/surveys/:surveyId/sms-sends`. **Apply JWT auth middleware**.
- **Task 6.11 (FE):** Implement `SmsFeedbackPage` (`apps/frontend/app/feedback/sms/[identifier]/page.js`) _(Server Component)_. Render `SurveyForm`. Use `(feedback)` layout.
- **Task 6.12 (FE):** Add `sendSms`, `getSmsSends` to `apiSurveys` client.
- **Task 6.13 (FE):** Create `SmsSendForm` component (`apps/frontend/components/surveys/SmsSendForm.jsx`) _(Client)_. Render on `SurveyDetailsDisplay`. **Uses `useMutation` (TanStack) calling `apiClient.sendSms`**. **Invalidate `smsSends` query on success**. Shows loading, uses `toast`.
- **Task 6.14 (FE):** Create `SmsSendList` component (`apps/frontend/components/surveys/SmsSendList.jsx`) _(Client)_. Render on `SurveyDetailsDisplay`. **Uses `useQuery` (TanStack, key: ['smsSends', surveyId]) calling `apiClient.getSmsSends`**. Displays in Table. Shows loading, uses `toast`.

- **End of Phase Check:** SMS functional, uses TanStack Query, invalidates cache on send.

---

**Phase 7: Incentive Mechanism (Discount Code)**

_Goal: Issue discount codes, verify/redeem via BE API + TanStack Query._

- **Task 7.1 (DB):** Define `DiscountCode` schema. Migrate & generate.
- **Task 7.2 (BE/Utils):** Implement `generateDiscountCode` utility.
- **Task 7.3 (BE/Service):** Modify `responseService` (SMS) to create `DiscountCode`. Modify public submit API response.
- **Task 7.4 (FE):** Modify `SurveyForm` mutation (SMS) to display code.
- **Task 7.5 (BE/Service):** Implement `discountService` (verify, redeem - check ownership via `req.user`).
- **Task 7.6 (BE/Controller):** Implement `discountController`.
- **Task 7.7 (BE/Routes):** Implement authenticated discount routes (`/api/v1/discounts/...`). **Apply JWT auth middleware**.
- **Task 7.8 (FE):** Add `apiDiscounts` client functions.
- **Task 7.9 (FE):** Implement `VerifyCodePage` (`apps/frontend/app/(app)/verify-code/page.js`) _(Server)_ rendering `VerifyCodeForm`.
- **Task 7.10 (FE):** Create `VerifyCodeForm` component (`apps/frontend/components/dashboard/VerifyCodeForm.jsx`) _(Client)_. **Uses `useQuery` (TanStack) for verification**. Displays status. **Uses `useMutation` (TanStack) for redemption**. **Invalidate verification query on success**. Shows loading, uses `toast`.

- **End of Phase Check:** Discount code lifecycle functional.

---

**Phase 8: Admin Panel**

_Goal: Implement admin functionalities using BE API + TanStack Query._

- **Task 8.1 (BE/Setup):** Create first ADMIN user in DB.
- **Task 8.2 (FE):** Create `(admin)` route group & layout (`apps/frontend/app/(admin)/layout.js`) _(Server)_. Implement client-side check or rely on BE protection.
- **Task 8.3 (BE/Middleware):** Implement/Enhance JWT middleware to include role checks (`isAdmin`).
- **Task 8.4 (BE/Service):** Implement `adminService` (user management).
- **Task 8.5 (BE/Controller):** Implement `adminController`.
- **Task 8.6 (BE/Routes):** Implement admin routes (`/api/v1/admin/...`). **Apply JWT auth AND admin role check middleware**.
- **Task 8.7 (FE):** Add `apiAdmin` client functions.
- **Task 8.8 (FE):** Implement `AdminDashboardPage` (`apps/frontend/app/(admin)/dashboard/page.js`) _(Server)_ rendering `PlatformStats`.
- **Task 8.9 (FE):** Create `PlatformStats` component (`apps/frontend/components/admin/PlatformStats.jsx`) _(Client)_. **Uses `useQuery` (TanStack) calling admin stats endpoint**. Shows loading, uses `toast`.
- **Task 8.10 (FE):** Implement `ManageUsersPage` (`apps/frontend/app/(admin)/users/page.js`) _(Server)_. Render `UserManagementTableLoader`.
- **Task 8.11 (FE):** Create `UserManagementTableLoader` component (`apps/frontend/components/admin/UserManagementTableLoader.jsx`) _(Client)_. **Uses `useQuery` (TanStack, key: ['adminUsers']) calling `apiClient.getAdminUsers`**. Handles loading/error. Renders `UserManagementTableDisplay`.
- **Task 8.12 (FE):** Create `UserManagementTableDisplay` component (`apps/frontend/components/admin/UserManagementTableDisplay.jsx`) _(Client)_. Receives `users` prop. Actions **use `useMutation` (TanStack) calling admin API endpoints**. Uses `AlertDialog`. **Invalidate `['adminUsers']` query on success**. Shows loading, uses `toast`.

- **End of Phase Check:** Admin panel functional, protected by BE middleware. Uses API routes and TanStack Query for data/mutations. Cache invalidation implemented.

---

**Phase 9: Final Touches & Deployment Prep**

_Goal: Prepare FE and BE for deployment, add deferred loading UI, finalize configurations._

- **Task 9.1 (BE):** Create/Finalize `Dockerfile` for backend (`apps/backend/Dockerfile`). Ensure Prisma generate/migrate steps are handled appropriately for production build/start.
- **Task 9.2 (Setup):** Finalize environment variables for FE (`.env.local`, `.env.example`) and BE (`.env`, `.env.example`). Document all required variables.
- **Task 9.3 (BE):** Configure backend CORS middleware strictly for production, using `process.env.CORS_ORIGIN` to allow only the production frontend domain(s).
- **Task 9.4 (FE):** Implement `loading.js` files in key FE route segments using `LoadingSpinner` or Shadcn `Skeleton`.
- **Task 9.5 (FE):** Review FE UI consistency, responsiveness.
- **Task 9.6 (General):** Perform thorough end-to-end manual testing.
- **Task 9.7 (Deployment):** Set up production PostgreSQL hosting.
- **Task 9.8 (Deployment):** Set up production Node.js hosting for backend. Configure env vars. Set up deployment pipeline (using `turbo build --filter=backend`, running migrations `npx prisma migrate deploy`).
- **Task 9.9 (Deployment):** Set up production Next.js hosting for frontend. Configure env vars. Set up deployment pipeline (using `turbo build --filter=frontend`).
- **Task 9.10 (Deployment):** Deploy both applications.
- **Task 9.11 (Deployment):** Test the fully deployed application end-to-end.

- **End of Phase Check:** Both applications deployed and functional. Page loading UI implemented. CORS restricted. Ready for V1 launch.

---
