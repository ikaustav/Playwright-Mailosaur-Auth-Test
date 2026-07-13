# Auth Flow CI Kit

Two things, both scoped to **an application you own**:

1. **`tests/signup-otp.spec.js`** — a Playwright regression test that signs
   up a fresh test account on your staging environment, retrieves the OTP
   from a Mailosaur test inbox you control, verifies it, and confirms the
   account reaches an authenticated dashboard. Also checks that a wrong OTP
   is correctly rejected.
2. **`docs/SECURITY_CHECKLIST.md`** — a checklist for hardening your own
   auth flow: rate limiting, OTP entropy/lifecycle, session fixation,
   clickjacking, and general signup/login hygiene.

## Setup

```bash
npm install
npx playwright install --with-deps chromium
cp .env.example .env.local   # fill in your own values
```

Required environment variables:

| Variable | Description |
|---|---|
| `BASE_URL` | Your staging/test app URL (e.g. `https://staging.yourapp.com`) |
| `MAILOSAUR_API_KEY` | API key for your Mailosaur account |
| `MAILOSAUR_SERVER_ID` | The Mailosaur server/inbox ID you use for test emails |
| `TEST_USER_PASSWORD` | Fixed password for the throwaway CI test accounts |

## Running locally

```bash
export $(cat .env.local | xargs)
npm test
```

## Running in CI

`.github/workflows/auth-flow-test.yml` runs the test on push to `main`, on a
daily schedule, and on manual dispatch. Add `STAGING_BASE_URL`,
`MAILOSAUR_API_KEY`, `MAILOSAUR_SERVER_ID`, and `TEST_USER_PASSWORD` as
repository secrets.

## Adjusting to your app

The test uses generic selectors (`getByLabel('Email')`,
`getByRole('button', { name: /sign up/i })`, etc.) — update these to match
your actual signup/verification UI. The OTP-extraction regex
(`/\b(\d{6})\b/`) assumes a 6-digit numeric code; adjust it to match your
email template if your codes look different.

## Out of scope (intentionally)

This kit does not discover or target arbitrary websites, generate synthetic
identities for evading anti-abuse systems, solve CAPTCHAs, or rotate proxies
to avoid detection. It's a regression test for one app you already control,
run with credentials and an inbox you already own. If you need broader
security testing against third-party systems, that requires a scoped,
written pentest authorization — not an automation script.
