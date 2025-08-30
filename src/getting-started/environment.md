# Environment Configuration

Hackcontrol uses environment variables for configuration. This guide covers all required and optional environment variables.

## Environment Variables

All environment variables are validated using Zod schemas in `src/env/index.mjs`.

### Required Variables

#### Database
```env
DATABASE_URL="postgresql://username:password@host:port/database?sslmode=require"
```
- **Purpose**: CockroachDB connection string
- **Format**: PostgreSQL connection URL with SSL
- **Example**: `postgresql://user:pass@cluster.cockroachdb.cloud:26257/hackcontrol?sslmode=require`

#### NextAuth Configuration
```env
NEXTAUTH_SECRET="your-secret-key-here"
NEXTAUTH_URL="http://localhost:3000"
```

- **NEXTAUTH_SECRET**: Random string for JWT signing (use `openssl rand -base64 32`)
- **NEXTAUTH_URL**: Your application's URL (production or development)

#### GitHub OAuth
```env
GITHUB_CLIENT_ID="your-github-app-client-id"
GITHUB_CLIENT_SECRET="your-github-app-secret"
```

- **GITHUB_CLIENT_ID**: Public identifier from your GitHub OAuth app
- **GITHUB_CLIENT_SECRET**: Secret key from your GitHub OAuth app

### Optional Variables

#### Node Environment
```env
NODE_ENV="development"  # or "production" or "test"
```

#### Vercel Deployment
```env
VERCEL_URL="your-app.vercel.app"  # Automatically set by Vercel
```

## Environment Validation

The project uses Zod for runtime environment validation:

```typescript
// Server-side validation
const server = z.object({
  DATABASE_URL: z.string().url(),
  NODE_ENV: z.enum(["development", "test", "production"]),
  NEXTAUTH_SECRET: z.string().min(1),
  NEXTAUTH_URL: z.string().url(),
  GITHUB_CLIENT_ID: z.string().min(1),
  GITHUB_CLIENT_SECRET: z.string().min(1),
});
```

## Creating Environment Files

### Development (.env)
```env
DATABASE_URL="postgresql://user:pass@localhost:26257/hackcontrol_dev?sslmode=disable"
NEXTAUTH_SECRET="dev-secret-key-change-in-production"
NEXTAUTH_URL="http://localhost:3000"
GITHUB_CLIENT_ID="your-dev-github-client-id"
GITHUB_CLIENT_SECRET="your-dev-github-client-secret"
NODE_ENV="development"
```

### Production (.env.production)
```env
DATABASE_URL="postgresql://user:pass@cluster.cockroachdb.cloud:26257/hackcontrol?sslmode=require"
NEXTAUTH_SECRET="super-secure-production-secret"
NEXTAUTH_URL="https://your-domain.com"
GITHUB_CLIENT_ID="your-prod-github-client-id"
GITHUB_CLIENT_SECRET="your-prod-github-client-secret"
NODE_ENV="production"
```

## Security Best Practices

1. **Never commit `.env` files** to version control
2. **Use different OAuth apps** for development and production
3. **Generate strong secrets** using cryptographically secure methods
4. **Rotate secrets regularly** in production
5. **Use environment-specific databases** to avoid data conflicts

## Generating Secrets

### NEXTAUTH_SECRET
```bash
# Using OpenSSL
openssl rand -base64 32

# Using Node.js
node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"
```

## Environment Validation Errors

Common validation errors and solutions:

- **Invalid URL format**: Ensure DATABASE_URL and NEXTAUTH_URL are properly formatted URLs
- **Missing required variables**: All variables in the `server` schema must be provided
- **Client-side access**: Server variables cannot be accessed on the client side

## Next Steps

- Set up your [Database](database.md)
- Configure [GitHub OAuth](../configuration/github-oauth.md)
- Learn about [Deployment Variables](../deployment/environment-vars.md)
