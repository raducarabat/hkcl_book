# Database Setup

Hackcontrol uses CockroachDB as its database, managed through Prisma ORM. This guide covers database setup and management.

## CockroachDB Setup

### 1. Create a Free Cluster

1. Visit [CockroachDB Cloud](https://www.cockroachlabs.com/get-started-cockroachdb/)
2. Sign up for a free account
3. Create a new cluster:
   - Choose "Serverless" for free tier
   - Select your preferred region
   - Name your cluster (e.g., "hackcontrol-dev")

### 2. Create Database User

1. In your cluster dashboard, go to "SQL Users"
2. Create a new user:
   - Username: `hackcontrol_user` (or your preference)
   - Generate a strong password
   - Save the credentials securely

### 3. Get Connection String

1. Click "Connect" on your cluster
2. Select "General connection string"
3. Copy the connection string
4. Replace `<username>` and `<password>` with your user credentials

Example connection string:
```
postgresql://hackcontrol_user:password@cluster-name.aws-region.cockroachlabs.cloud:26257/hackcontrol?sslmode=require
```

## Database Schema

The application uses the following main models:

### Core Models
- **Hackathon**: Main hackathon events
- **Participation**: User project submissions
- **User**: User accounts and profiles
- **Judge**: Judging assignments
- **Score**: Judging scores
- **Announcement**: Event announcements

### Authentication Models
- **Account**: OAuth account linking
- **Session**: User sessions
- **VerificationToken**: Email verification

## Prisma Migration

### Initial Setup

Run the initial migration to create all tables:

```bash
npx prisma migrate dev --name init
```

### Schema Changes

When you modify `prisma/schema.prisma`:

```bash
# Create and apply migration
npx prisma migrate dev --name describe-your-change

# Generate Prisma client
npx prisma generate
```

### Useful Commands

```bash
# View database in Prisma Studio
npm run studio

# Reset database (development only)
npx prisma migrate reset

# Deploy migrations (production)
npx prisma migrate deploy

# Check migration status
npx prisma migrate status
```

## Database Administration

### Setting Admin Role

After your first login, grant admin privileges:

1. Open CockroachDB console or Prisma Studio
2. Navigate to the `users` table
3. Run this SQL command:

```sql
UPDATE users SET role = 'ADMIN' WHERE email = 'your-github-email@example.com';
```

Or using Prisma Studio:
1. Open `npm run studio`
2. Navigate to `User` model
3. Find your user record
4. Change `role` from `USER` to `ADMIN`
5. Save changes

### User Roles

The application supports three user roles:

- **USER**: Default role, can participate in hackathons
- **ORGANIZER**: Can create and manage hackathons
- **ADMIN**: Full system access, can manage all hackathons and users

## Backup and Recovery

### Export Data
```bash
# Export schema
npx prisma db pull

# Export data (using pg_dump)
pg_dump $DATABASE_URL > backup.sql
```

### Restore Data
```bash
# Restore from backup
psql $DATABASE_URL < backup.sql
```

## Troubleshooting

### Connection Issues
- Verify connection string format
- Check firewall settings
- Ensure SSL mode is set correctly

### Migration Errors
- Check for schema conflicts
- Verify database permissions
- Review migration history

### Performance
- Monitor query performance in CockroachDB console
- Use Prisma query logging for development
- Consider database indexing for large datasets

## Next Steps

- Learn about [Prisma ORM](../technologies/prisma.md)
- Explore [Database Schema](../architecture/database-schema.md)
- Set up [Development Workflow](../development/workflow.md)
