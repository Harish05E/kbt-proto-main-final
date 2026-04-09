# ThermoSmart Production Launch QA Suite

## 1. Functional Test Cases

| ID | Description | Preconditions | Steps | Expected Result | Edge Cases |
|---|---|---|---|---|---|
| AUTH-001 | Login success | User on /login | Enter valid email/password, click Sign In | Redirect to /dashboard and user context set | Refresh after login should keep session |
| AUTH-002 | Login empty fields | User on /login | Submit with one or both fields empty | Inline error shown, no redirect | Whitespace-only input |
| AUTH-003 | Signup success | User on /signup | Fill all fields, accept terms, submit | Redirect to /dashboard, user persisted | Long company name |
| AUTH-004 | Signup password mismatch | User on /signup | Enter non-matching password and confirm | Password mismatch error shown | Re-typing confirm field clears error |
| AUTH-005 | Logout flow | Authenticated user on /dashboard | Click profile menu, click Logout | Redirect to /login and user cleared | Double-click logout |
| FORM-001 | Email validation | /login or /signup open | Use invalid email format and submit | Validation error shown | Missing domain, unicode email |
| FORM-002 | Required fields | /signup open | Leave each required field empty in turn | Proper field-specific or form-level error | Clipboard pasted spaces |
| NAV-001 | Protected route redirect | User not authenticated | Open /dashboard directly | Redirect to /login | Back button after redirect |
| NAV-002 | Sidebar route transitions | Authenticated dashboard user | Click each sidebar link | Correct page renders, active state updates | Repeated rapid clicking |
| API-001 | API success response | Mock API active | Trigger data-fetch action | Success state and data shown | Empty list response |
| API-002 | API 4xx handling | Mock API returns 400/401 | Trigger request | User-friendly error message shown | Auth-expired should force re-login |
| API-003 | API 5xx handling | Mock API returns 500 | Trigger request | Retry option or fallback shown | Persisted partial state |
| API-004 | API timeout | Mock delayed response | Trigger request with timeout threshold | Timeout message + graceful recovery | Retry on unstable network |
| STATE-001 | Auth state hydration | localStorage has valid user | Load app | User restored after mount | Corrupted JSON in localStorage |
| STATE-002 | Cart quantity updates | Cart has one item | Update quantity and recalc | Total price updates accurately | Quantity 0 or negative |
| ERR-001 | Runtime error boundary | Inject UI render error | Open affected route | Error fallback shown, app not frozen | Error in nested child component |

## 2. UI/UX Testing Matrix

- Responsiveness: 360x800, 768x1024, 1024x1366, 1440x900, 1920x1080.
- Browsers: Chrome latest, Edge latest, Firefox latest, Safari 17+.
- Accessibility checks:
  - Keyboard navigation from header to CTA and forms.
  - All form controls have associated labels.
  - Visible focus states and no keyboard traps.
  - Minimum contrast ratio 4.5:1 for body text.
- Loading states:
  - Spinner visible during auth transitions.
  - Skeletons required for dashboard cards with API-backed data.
- Layout integrity:
  - No horizontal overflow at 320px width.
  - Dropdown and sidebar overlays never clipped.
- Interactive states:
  - Buttons show hover/active/disabled styles consistently.
  - Disabled submit buttons cannot be triggered by keyboard.

## 3. Performance Testing Plan

- Lighthouse targets:
  - Performance >= 90
  - Accessibility >= 95
  - Best Practices >= 95
  - SEO >= 95
- Core Web Vitals targets:
  - LCP < 2.5s
  - INP < 200ms
  - CLS < 0.1
- Optimization actions:
  - Convert brand and decorative visuals to optimized SVG and use next/image for raster media.
  - Lazy-load heavy dashboard submodules using dynamic imports.
  - Memoize derived lists and expensive computations with useMemo.
  - Wrap stable handlers in useCallback where prop drilling triggers rerenders.
  - Use request deduping/caching via fetch cache controls or React Query/SWR.

## 4. Backend and API Testing

- Validate schema on every response:
  - Success shape: `{ data, meta? }`
  - Error shape: `{ error: { code, message, details? } }`
- Status code matrix:
  - 200/201 success
  - 400 validation
  - 401 unauthenticated
  - 403 unauthorized
  - 404 not found
  - 429 rate limited
  - 500 internal error
- Failure simulations:
  - Inject 2s/5s/15s delays.
  - Return malformed JSON.
  - Return stale token response and verify forced logout.

## 5. Security Testing

- XSS:
  - Attempt script injection in every text input and query string.
- CSRF:
  - Ensure state-changing requests require CSRF token or same-site protections.
- Input sanitization:
  - Enforce server-side validation with strict schema (zod recommended).
- Token/cookie storage:
  - Prefer httpOnly + secure + sameSite=strict cookies over localStorage.
- Secrets hygiene:
  - Ensure no NEXT_PUBLIC variables contain secrets.
  - Audit build output for leaked tokens.

## 6. Edge and Negative Testing

- Network offline while submitting login/signup.
- Slow 3G mode for loading states and timeout messaging.
- Multi-tab scenario with logout in one tab and protected route in another.
- Browser refresh during async transitions.
- Invalid deep links under /dashboard subroutes.

## 7. Automated Test Setup

### Suggested Structure

```text
tests/
  e2e/
    auth.spec.ts
    navigation.spec.ts
  unit/
    auth-context.test.tsx
    cart-context.test.tsx
playwright.config.ts
jest.config.ts
jest.setup.ts
```

### Commands

- `pnpm test:e2e`
- `pnpm test:unit`
- `pnpm test:ci`

## 8. Vercel Deployment Checklist

- Environment variables validated in Vercel Project Settings.
- `pnpm build` passes with zero TypeScript and lint errors.
- No missing route handlers or 404 regressions for app router paths.
- Metadata includes title/description/icons and social tags.
- Analytics configured and verified (`@vercel/analytics`).
- Preview deploy smoke test for login, signup, dashboard, and logout.

## 9. Branding and v0 Logo Removal

Completed in app code:

- Custom favicon at `/public/favicon.svg`
- Shared brand logo component in `components/brand-logo.tsx`
- Metadata icon refs updated in `app/layout.tsx`
- Legacy `generator: 'v0.app'` removed
- Header/auth/dashboard branding switched to `ThermoSmart`
