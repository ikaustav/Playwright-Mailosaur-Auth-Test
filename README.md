# 🛡️ Playwright + Mailosaur Auth-Flow Test Kit

[![Playwright](https://img.shields.io/badge/Playwright-Enabled-2EAD33?logo=playwright)](https://playwright.dev/)
[![Mailosaur](https://img.shields.io/badge/Mailosaur-Integrated-blue?logo=mailosaur)](https://mailosaur.com/)

A complete, automated testing kit designed to regression-test your application's authentication flow in Continuous Integration (CI). This suite validates your end-to-end signup process, from initial registration to OTP email retrieval, verification, and dashboard access.

> **⚠️ Important Notice:** This is a working kit strictly intended for testing **your own application**. It does not discover or target arbitrary sites, and there is no vulnerability scanning component included. It relies on a base URL and a Mailosaur inbox that you actively own and configure.

---

## ✨ Features

*   **End-to-End Auth Testing:** Automates the complete user journey (Signup → Receive OTP → Verify → Authenticated Dashboard).
*   **Mailosaur Integration:** Programmatically polls a dedicated testing inbox to intercept and read OTP emails in real time.
*   **Negative Testing:** Automatically confirms that incorrect or expired codes are properly rejected.
*   **CI/CD Ready:** Includes a GitHub Actions workflow out-of-the-box for automated runs on push, schedule, or manual dispatch.
*   **Security Hardening Guide:** Ships with a robust security checklist for your development team to review auth-specific vulnerabilities.

---

## 📂 Project Structure

```text
.
├── .github/
│   └── workflows/
│       └── auth-flow-test.yml    # GitHub Actions CI pipeline
├── docs/
│   └── SECURITY_CHECKLIST.md     # Auth-flow defensive security checklist
├── tests/
│   └── signup-otp.spec.js        # Main Playwright test suite
├── .env.example                  # Example environment variables
├── playwright.config.js          # Playwright configuration file
├── package.json                  # Node dependencies and scripts
└── README.md                     # Project documentation

🛠️ Prerequisites
Before you begin, ensure you have the following:

Node.js: v16.x or higher installed on your machine.

Mailosaur Account: An active account with an API key and a configured Server ID.

Target App Environment: A staging or local development version of your application to test against.

🚀 Installation & Setup
1. Clone the repository:

Bash
git clone [https://github.com/your-username/auth-flow-test-kit.git](https://github.com/your-username/auth-flow-test-kit.git)
cd auth-flow-test-kit
2. Install Node dependencies:

Bash
npm install
3. Install Playwright Browsers:

Bash
npx playwright install --with-deps chromium
4. Configure Environment Variables:
Copy the example environment file and fill in your credentials.

Bash
cp .env.example .env.local
Open .env.local and configure the following variables:

Code snippet
BASE_URL=[https://staging.yourapp.com](https://staging.yourapp.com)
MAILOSAUR_API_KEY=your_mailosaur_api_key_here
MAILOSAUR_SERVER_ID=your_server_id_here
5. Update Test Selectors:
Open tests/signup-otp.spec.js and update the DOM selectors (e.g., #email-input, #submit-btn) to match your application's actual HTML structure.

🧪 Running the Tests
You can run the tests using various Playwright commands depending on your needs:

Run tests in headless mode (Default):

Bash
npm test
Run tests with the Playwright UI (Recommended for debugging):

Bash
npm run test:ui
Run tests in headed mode (Watch the browser execute):

Bash
npm run test:headed
⚙️ CI/CD Integration
This kit includes a fully configured GitHub Actions workflow (.github/workflows/auth-flow-test.yml).

To activate it:

Navigate to your repository's Settings > Secrets and variables > Actions.

Add the following repository secrets:

 ~BASE_URL
 ~MAILOSAUR_API_KEY
 ~MAILOSAUR_SERVER_ID

The tests will now automatically run on any push to the main branch, or via a nightly schedule.

🔐 Security Checklist
Auth flows are frequent targets for exploitation. Please review the included docs/SECURITY_CHECKLIST.md with your development team. It covers critical defensive checks, including:

~Rate Limiting & Throttling: Preventing brute-force attacks on OTP endpoints.
~OTP Lifecycle & Entropy: Ensuring codes are sufficiently random and expire promptly.
~Session Management: Preventing session fixation post-authentication.
~UI Defenses: Validating CORS policies and clickjacking mitigations (X-Frame-Options / CSP).

📄 License
This project is licensed under the MIT License - see the LICENSE file for details.
