# Local Installation

Complete guide for setting up Hackcontrol on your local development environment.

## System Requirements

- **Node.js**: Version 18.0 or higher
- **npm**: Version 8.0 or higher (or pnpm/yarn)
- **Git**: Latest version
- **Database**: CockroachDB (cloud or local)

## Step-by-Step Installation

### 1. Clone the Repository

```bash
git clone https://github.com/raducarabat/hackcontrol.git
cd hackcontrol
```

### 2. Install Dependencies

Choose your preferred package manager:

```bash
# Using npm
npm install

# Using pnpm (recommended for faster installs)
pnpm install

# Using yarn
yarn install
```

### 3. Environment Configuration

Copy the example environment file and configure it:

```bash
cp .env.example .env  # If available, or create new .env
```

Edit `.env` with your configuration:

```env
# CockroachDB connection string
DATABASE_URL="postgresql://username:password@host:port/database?sslmode=require"

# NextAuth configuration
NEXTAUTH_SECRET="your-generated-secret-here"
NEXTAUTH_URL="http://localhost:3000"

# GitHub OAuth Provider
GITHUB_CLIENT_ID="your-github-app-client-id"
GITHUB_CLIENT_SECRET="your-github-app-client-secret"
```

### 4. Database Migration

Apply the database schema:

```bash
npx prisma migrate dev --name init
```

This command will:
- Create migration files in `prisma/migrations/`
- Apply the schema to your database
- Generate the Prisma client

### 5. Set Admin Privileges

After your first login, grant yourself admin access:

1. Sign in through the application once
2. Connect to your CockroachDB console
3. Run this SQL command:

```sql
UPDATE users SET role = 'ADMIN' WHERE email = 'your-github-email@example.com';
```

### 6. Start Development Server

```bash
npm run dev
```

The application will be available at [http://localhost:3000](http://localhost:3000).

## Verification

To verify your installation:

1. **Server starts**: No errors in terminal
2. **Database connected**: Check Prisma Studio with `npm run studio`
3. **Authentication works**: Can sign in with GitHub
4. **Admin access**: Can create new hackathons

## Common Issues

- **Database connection errors**: Check your `DATABASE_URL` format
- **Authentication fails**: Verify GitHub OAuth app configuration
- **Migration errors**: Ensure database exists and is accessible
- **Port conflicts**: Change port with `npm run dev -- -p 3001`

## Next Steps

- Configure your [Environment Variables](environment.md)
- Set up your [Database](database.md) in detail
- Learn about the [Project Architecture](../architecture/overview.md)
