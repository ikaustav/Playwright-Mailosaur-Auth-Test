#!/bin/bash

echo "🚀 Bootstrapping Playwright + Mailosaur Auth-Flow Test Kit..."

# Create directory structure
mkdir -p .github/workflows docs tests

# 1. package.json
cat << 'EOF' > package.json
{
  "name": "auth-flow-test-kit",
  "version": "1.0.0",
  "description": "Playwright and Mailosaur automated auth flow tests",
  "scripts": {
    "test": "playwright test",
    "test:ui": "playwright test --ui",
    "test:headed": "playwright test --headed"
  },
  "dependencies": {
    "mailosaur": "^8.5.0"
  },
  "devDependencies": {
    "@playwright/test": "^1.40.0",
    "dotenv": "^16.3.1"
  }
}
EOF

# 2. playwright.config.js
cat << 'EOF' > playwright.config.js
const { defineConfig, devices } = require('@playwright/test');
require('dotenv').config({ path: '.env.local' });

module.exports = defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: process.env.BASE_URL,
    trace: 'on-first-retry',
  },
  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    }
  ],
});
EOF

# 3. .env.example
cat << 'EOF' > .env.example
# Your target staging or local URL
BASE_URL=https://staging.yourapp.com

# Mailosaur Credentials
MAILOSAUR_API_KEY=your_mailosaur_api_key_here
MAILOSAUR_SERVER_ID=your_server_id_here
EOF

# 4. tests/signup-otp.spec.js
cat << 'EOF' > tests/signup-otp.spec.js
const { test, expect } = require('@playwright/test');
const MailosaurClient = require('mailosaur');

test.describe('Authentication Flow Validation', () => {
  const mailosaur = new MailosaurClient(process.env.MAILOSAUR_API_KEY);
  const serverId = process.env.MAILOSAUR_SERVER_ID;

  test('User can sign up, receive OTP, and authenticate', async ({ page }) => {
    // Generate a random email address using your Mailosaur Server ID
    const randomString = Math.random().toString(36).substring(7);
    const testEmail = `test-${randomString}@${serverId}.mailosaur.net`;

    // 1. Navigate to your app and initiate signup
    await page.goto('/signup'); // Appends to BASE_URL automatically
    await page.fill('input[name="email"]', testEmail);
    await page.fill('input[name="password"]', 'SecurePassword123!');
    await page.click('button[type="submit"]');

    // 2. Poll Mailosaur for the OTP email
    const email = await mailosaur.messages.get(serverId, {
      sentTo: testEmail
    });

    // Extract OTP (Assumes OTP is a 6-digit code in the email body)
    const otpMatch = email.text.body.match(/\b\d{6}\b/);
    expect(otpMatch).toBeTruthy();
    const otpCode = otpMatch[0];

    // 3. Enter the OTP into your application
    await page.fill('input[name="otp"]', otpCode);
    await page.click('button[type="submit"]');

    // 4. Verify successful authentication (e.g., landing on dashboard)
    await expect(page).toHaveURL(/.*\/dashboard/);
    await expect(page.locator('text="Welcome"')).toBeVisible();
  });
});
EOF

# 5. .github/workflows/auth-flow-test.yml
cat << 'EOF' > .github/workflows/auth-flow-test.yml
name: Auth Flow CI
on:
  push:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * *' # Runs nightly at 2 AM
  workflow_dispatch:

jobs:
  test:
    timeout-minutes: 10
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    
    - name: Setup Node.js
      uses: actions/setup-node@v4
      with:
        node-version: lts/*
        
    - name: Install dependencies
      run: npm ci
      
    - name: Install Playwright Browsers
      run: npx playwright install --with-deps chromium
      
    - name: Run Playwright tests
      run: npm run test
      env:
        BASE_URL: ${{ secrets.BASE_URL }}
        MAILOSAUR_API_KEY: ${{ secrets.MAILOSAUR_API_KEY }}
        MAILOSAUR_SERVER_ID: ${{ secrets.MAILOSAUR_SERVER_ID }}
        
    - uses: actions/upload-artifact@v4
      if: always()
      with:
        name: playwright-report
        path: playwright-report/
        retention-days: 15
EOF

# 6. docs/SECURITY_CHECKLIST.md
cat << 'EOF' > docs/SECURITY_CHECKLIST.md
# 🔐 Auth Flow Security Checklist

Use this checklist to harden your authentication endpoints and protect against common exploits.

## 1. Brute Force & Rate Limiting
- [ ] Are IP-based rate limits enforced on the `/signup` and `/login` endpoints?
- [ ] Are account-based rate limits enforced for OTP verification?
- [ ] Does the system temporarily lock out an account after X failed OTP attempts?

## 2. OTP Lifecycle & Entropy
- [ ] Are OTP codes cryptographically random?
- [ ] Do OTP codes expire within a short timeframe (e.g., 5-15 minutes)?
- [ ] Is an OTP immediately invalidated once it is successfully used?

## 3. Session Management
- [ ] Is a new session ID generated upon successful OTP verification to prevent Session Fixation?
- [ ] Are authentication cookies flagged with `HttpOnly`, `Secure`, and `SameSite` attributes?

## 4. UI / Transport Defenses
- [ ] Are X-Frame-Options or Content-Security-Policy (CSP) headers set to prevent clickjacking?
- [ ] Is HSTS (Strict-Transport-Security) enabled across the site?
EOF

# 7. README.md
cat << 'EOF' > README.md
# 🛡️ Playwright + Mailosaur Auth-Flow Test Kit

A complete, automated testing kit designed to regression-test your application's authentication flow in Continuous Integration (CI). This suite validates your end-to-end signup process, from initial registration to OTP email retrieval, verification, and dashboard access.

## 🚀 Quick Start
1. Run `npm install`
2. Run `npx playwright install --with-deps chromium`
3. Copy `.env.example` to `.env.local` and fill in your keys.
4. Run tests with `npm test`
EOF

echo "✅ Setup complete! Navigate to the directory, run 'npm install', and setup your '.env.local' file."
