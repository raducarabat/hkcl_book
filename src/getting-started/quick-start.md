# Quick Start Guide

This guide will get you up and running with Hackcontrol in under 10 minutes.

## Prerequisites

- Node.js 18+ installed
- Git installed
- CockroachDB account (free tier available)
- GitHub account for OAuth

## 1. Clone and Install

```bash
git clone https://github.com/raducarabat/hackcontrol.git
cd hackcontrol
npm install
```

## 2. Environment Setup

Create a `.env` file in the root directory:

```env
# Database
DATABASE_URL="your-cockroachdb-connection-string"

# NextAuth
NEXTAUTH_SECRET="your-secret-key"
NEXTAUTH_URL="http://localhost:3000"

# GitHub OAuth
GITHUB_CLIENT_ID="your-github-client-id"
GITHUB_CLIENT_SECRET="your-github-client-secret"
```

## 3. Database Setup

```bash
npx prisma migrate dev --name init
```

This creates the initial migration and applies it to your database.

## 4. Run Development Server

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000) in your browser.

## 5. First Login & Admin Setup

1. Click "Sign in with GitHub"
2. Authorize the application
3. **Important:** Set your account as admin by running this SQL command in your CockroachDB console:

```sql
UPDATE users SET role = 'ADMIN' WHERE email = 'your-email@example.com';
```

Replace `your-email@example.com` with the email associated with your GitHub account.

4. Refresh the page - you now have full admin privileges!

## Next Steps

- Read the [Local Installation](installation.md) guide for detailed setup
- Learn about [Environment Configuration](environment.md)
- Explore the [Project Structure](../architecture/project-structure.md)
