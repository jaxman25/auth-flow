# MERNAUTH — MERN Authentication Starter

MERNAUTH is a reusable authentication starter for applications built with the MERN stack. It separates the user-facing frontend, admin frontend, and backend API.

## Features

The project documentation describes the following features. Confirm each one against the current implementation before using the project in production.

- User-facing React application
- Admin dashboard built with React
- Node.js and Express API backed by MongoDB
- JWT authentication and database-backed sessions
- Optional HTTP-only cookie authentication
- Email verification using a code
- Password reset by email
- Google OAuth sign-in/sign-up
- Protected routes and login rate limiting
- User profile management

## Project Structure

```text
MERNAUTH/
├── admin/       # Admin dashboard frontend
├── user/        # User-facing frontend
├── backend/     # Express API and MongoDB integration
├── package.json # Root scripts and shared tooling
└── README.md
```

> Folder names are case-sensitive on some systems. If your actual folder is named `User` rather than `user`, use that exact name in commands and scripts.

## Requirements

Install the following before getting started:

- Node.js and npm
- MongoDB locally, or a MongoDB Atlas account
- Google OAuth credentials if you want Google sign-in
- A Gmail account with a Google App Password if the backend sends email through Gmail

Check your installed versions:

```bash
node --version
npm --version
```

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/jaxman25/auth-flow
cd auth-flow
```

### 2. Install dependencies

Install the root dependencies:

```bash
npm install
```

Then install dependencies in each application if the root setup does not install them automatically:

```bash
cd backend
npm install
cd ../user
npm install
cd ../admin
npm install
cd ..
```

If your folder is named `User` with a capital `U`, change `cd ../user` to `cd ../User`.

### 3. Configure MongoDB

Choose one option.

**Option A — MongoDB Atlas**

1. Create a cluster in [MongoDB Atlas](https://www.mongodb.com/atlas).
2. Create a database user and allow access from your development IP address.
3. Copy the connection string and replace its placeholders.

Example:

```env
MONGO_URI=mongodb+srv://<username>:<password>@<cluster-host>/mernauth
```

**Option B — Local MongoDB**

Install and start MongoDB locally, then use a connection string such as:

```env
MONGO_URI=mongodb://127.0.0.1:27017/mernauth
```

### 4. Configure environment variables

Create a file named `.env` inside `backend/`. Use the following as a template and adjust the values and variable names to match the backend code:

```env
# Server
PORT=5000
NODE_ENV=development

# MongoDB
MONGO_URI=mongodb://127.0.0.1:27017/mernauth

# JWT
JWT_SECRET=replace_with_a_long_random_secret
JWT_EXPIRES_IN=7d

# Frontend URLs
CLIENT_URL=http://localhost:5173
ADMIN_CLIENT_URL=http://localhost:5174

# Google OAuth (if enabled)
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret

# Gmail email delivery (if enabled)
GMAIL_USER=your_email@gmail.com
GMAIL_APP_PASSWORD=your_google_app_password
```

Important:

- `CLIENT_URL` and `ADMIN_CLIENT_URL` must match the actual frontend URLs and the backend's CORS configuration.
- Create OAuth credentials in the [Google Cloud Console](https://console.cloud.google.com/). Configure the authorized origins and redirect URIs required by your implementation.
- `GMAIL_APP_PASSWORD` refers to a Google App Password, not your normal Gmail password. App Passwords require eligible Google Account security settings.
- Do not commit `.env` files, real credentials, JWT secrets, or database passwords to Git.
- If the code uses additional environment variables, add them to your local `.env` file and document them here.

## Running the Applications

The commands below assume each package defines a `dev` script and the root package defines a `start` script. Check the relevant `package.json` files if a command is unavailable.

### Start all applications

From the project root:

```bash
npm start
```

The root `package.json` can use `concurrently` to run the services together. For example, if the package scripts match this project:

```json
{
  "scripts": {
    "start": "concurrently \"npm run dev --prefix backend\" \"npm run dev --prefix user\" \"npm run dev --prefix admin\""
  }
}
```

Install `concurrently` at the root if that script uses it:

```bash
npm install --save-dev concurrently
```

### Start each application separately

Open a terminal for each service.

**Backend API**

```bash
cd backend
npm run dev
```

**User frontend**

```bash
cd user
npm run dev
```

**Admin frontend**

```bash
cd admin
npm run dev
```

Use `User` instead of `user` if that is the actual directory name.

The example development URLs are:

| Service | Example URL |
|---|---|
| Backend API | `http://localhost:5000` |
| User frontend | `http://localhost:5173` |
| Admin frontend | `http://localhost:5174` |

These are examples only. The actual ports depend on your server and frontend configuration.

## Authentication API

The following routes are described by the project documentation. Confirm the route paths, HTTP methods, and response formats against the backend router files.

| Method | Endpoint | Purpose |
|---|---|---|
| `POST` | `/api/auth/signup` | Register a user and initiate email verification |
| `POST` | `/api/auth/verify-email` | Verify an email using a code |
| `POST` | `/api/auth/login` | Authenticate with email/username and password |
| `POST` | `/api/auth/google` | Sign in or register with Google |
| `POST` | `/api/auth/logout` | End the current session |
| `GET` | `/api/auth/profile` | Get the authenticated user's profile |
| `PUT` | `/api/auth/profile` | Update profile information |
| `PUT` | `/api/auth/profile/password` | Change the password |
| `POST` | `/api/auth/forgot-password` | Request a password-reset email |
| `POST` | `/api/auth/reset-password/:token` | Reset a password using a reset token |

## User Application

The `user/` frontend provides the user-facing experience. The documented pages and flows include:

- **Sign up:** collect the required account details and initiate email verification.
- **Email verification:** submit the code sent to the user's email address.
- **Login:** sign in with the credentials supported by the backend.
- **Forgot/reset password:** request a reset email and set a new password.
- **Profile:** view and update account details and change the password.
- **Dashboard:** an example protected area for signed-in users.

The documented frontend stack includes React, React Router, Axios, Bootstrap, and `@react-oauth/google`. Verify the actual dependencies in `user/package.json`.

If the frontend calls the backend directly, configure its API base URL to match your environment. For example:

```js
axios.defaults.baseURL = "http://localhost:5000";
```

Prefer a frontend environment variable for the API URL when the project supports it, rather than hard-coding a development URL.

## Admin Application

The `admin/` frontend is intended for administrative tasks, such as:

- Managing users
- Viewing and updating admin profile settings
- Searching user records
- Disabling or banning accounts, if implemented
- Reviewing sessions or login history, if implemented
- Managing system settings, if implemented

Start the admin frontend and open its configured local URL. Administrative API endpoints should be protected by backend authorization checks; hiding admin pages in the frontend alone is not sufficient.

### Admin account setup

The project documentation refers to a `seedAdmin` utility, but the exact file path and default credentials must be confirmed in the repository.

- Inspect the backend's seeder or documented admin-creation process.
- Do not assume default credentials unless they are explicitly provided by the project.
- Change any seeded password immediately.
- Do not expose default admin credentials in a public repository or production environment.

## Email Delivery

The backend documentation describes Nodemailer with Gmail. A typical transport configuration looks like this:

```js
import nodemailer from "nodemailer";

const transporter = nodemailer.createTransport({
  service: "gmail",
  auth: {
    user: process.env.GMAIL_USER,
    pass: process.env.GMAIL_APP_PASSWORD,
  },
});
```

Example helper:

```js
export const sendEmail = async (toEmail, subject, text, html) => {
  const mailOptions = {
    from: `"MERNAUTH" <${process.env.GMAIL_USER}>`,
    to: toEmail,
    subject,
    text,
    html,
  };

  return transporter.sendMail(mailOptions);
};
```

This is an example, not a replacement for the existing mail module. Keep the implementation consistent with your project's module system and error-handling conventions.

## How Authentication Works

The documented authentication design uses:

- **JWTs** to represent authenticated users
- **Database-backed sessions** to track valid logins
- **Cookies**, if configured, to store tokens in HTTP-only cookies
- **Email verification** to confirm account ownership
- **Password-reset tokens** that should be securely generated, hashed before storage, and expired
- **Protected backend routes** that verify authentication and authorization

A session-backed design should invalidate a session when a user logs out. If the app supports a single active session per user, verify that the backend enforces this behavior.

### Cookie example

If cookie authentication is implemented, a configuration may look like this:

```js
res.cookie("token", token, {
  httpOnly: true,
  secure: process.env.NODE_ENV === "production",
  sameSite: "strict",
  maxAge: 7 * 24 * 60 * 60 * 1000,
});
```

The correct `sameSite`, `secure`, CORS, and credentials settings depend on your deployment architecture. Test them in the target environment. For Axios requests that need cookies:

```js
axios.get("/api/auth/profile", { withCredentials: true });
```

Do not store sensitive session credentials in browser storage unless you have deliberately assessed the security trade-offs. The frontend and backend must use a consistent authentication strategy.

## Security Checklist

Before deploying publicly, verify that the implementation:

- Hashes passwords with a suitable password-hashing library such as bcrypt.
- Uses a strong, private JWT secret and validates token expiry.
- Validates the session on protected requests and invalidates it on logout.
- Applies rate limits to login and other sensitive endpoints.
- Generates unpredictable password-reset tokens, stores only a hash where appropriate, and enforces expiration and one-time use.
- Validates and sanitizes user input.
- Enforces authorization for administrative endpoints on the backend.
- Configures CORS, cookies, HTTPS, and allowed redirect URLs for the deployment environment.
- Keeps credentials and `.env` files out of version control.
- Handles authentication errors without exposing secrets or unnecessary account details.

A README description is not proof that a security control is implemented. Review and test the actual code before relying on any of these protections.

## Manual Testing Checklist

Use this checklist after configuring the project:

- [ ] Sign up with valid details.
- [ ] Confirm that the verification email arrives and the code works.
- [ ] Confirm that an unverified account is handled as intended.
- [ ] Log in using the supported email/username options.
- [ ] Try an incorrect password and verify error and rate-limit behavior.
- [ ] Request a password reset and test the complete reset flow.
- [ ] Test Google sign-in if configured.
- [ ] Open protected profile and admin endpoints while signed out.
- [ ] Log out and confirm the session can no longer be used.
- [ ] Test profile updates and password changes.
- [ ] Verify admin permissions using a non-admin account.
- [ ] Check browser/server logs for configuration or runtime errors.

## Customization

You can adapt MERNAUTH to your application by updating:

- Frontend branding, colors, typography, and layout
- User profile fields and account settings
- Admin dashboard pages and permissions
- Email templates and sender configuration
- Backend validation and business logic
- Environment-specific API URLs

Keep frontend behavior, backend validation, and database models consistent when adding features.

## License

Check the repository for a `LICENSE` file and follow its terms. If no license is present, do not assume the code is licensed for unrestricted reuse.

---

**In short:** MERNAUTH is intended to provide a reusable starting point for a MERN authentication system. Install the dependencies, configure MongoDB and environment variables, run the apps, and test the authentication flows and security controls before deploying.
