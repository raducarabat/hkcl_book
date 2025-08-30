# NextAuth.js Authentication

NextAuth.js provides secure, flexible authentication for Hackcontrol with GitHub OAuth integration.

## Authentication Setup

### Configuration

```typescript
// src/lib/auth.ts
export const authOptions: NextAuthOptions = {
  // Session strategy
  session: {
    strategy: "jwt",
  },
  
  // Database adapter
  adapter: PrismaAdapter(prisma),
  
  // OAuth providers
  providers: [
    GithubProvider({
      clientId: env.GITHUB_CLIENT_ID,
      clientSecret: env.GITHUB_CLIENT_SECRET,
      profile(profile) {
        return {
          id: profile.id.toString(),
          name: profile.name || profile.login,
          username: profile.login,
          email: profile.email,
          image: profile.avatar_url,
        };
      },
    }),
  ],
  
  // Custom pages
  pages: {
    signIn: "/auth",
  },
};
```

### API Route

```typescript
// src/pages/api/auth/[...nextauth].ts
import { authOptions } from "@/lib/auth";
import NextAuth from "next-auth";

export default NextAuth(authOptions);
```

## Authentication Flow

### 1. User Access
```
User visits protected page
       ↓
Check session status
       ↓
No session? → Redirect to /auth
       ↓
Session exists? → Allow access
```

### 2. GitHub OAuth Flow
```
Click "Sign in with GitHub"
       ↓
Redirect to GitHub OAuth
       ↓
User authorizes application
       ↓
GitHub redirects back with code
       ↓
NextAuth exchanges code for user data
       ↓
Create/update user in database
       ↓
Generate JWT session token
       ↓
Set session cookie
```

## Session Management

### JWT Strategy

```typescript
// JWT callbacks
callbacks: {
  async jwt({ token, user }) {
    const dbUser = await prisma.user.findFirst({
      where: { email: token.email },
    });
    
    if (!dbUser) {
      token.id = user?.id;
      token.role = "USER";
      return token;
    }
    
    return {
      id: dbUser.id,
      name: dbUser.name,
      username: dbUser.username,
      email: dbUser.email,
      image: dbUser.image,
      role: dbUser.role || "USER",
    };
  },
  
  async session({ token, session }) {
    if (token) {
      session.user.id = token.id;
      session.user.name = token.name;
      session.user.username = token.username;
      session.user.email = token.email;
      session.user.image = token.image;
      session.user.role = token.role;
    }
    return session;
  },
},
```

### Server-Side Session Access

```typescript
// In API routes or getServerSideProps
export const getServerAuthSession = async (ctx: {
  req: GetServerSidePropsContext["req"];
  res: GetServerSidePropsContext["res"];
}) => {
  return await getServerSession(ctx.req, ctx.res, authOptions);
};

// Usage in pages
export const getServerSideProps: GetServerSideProps = async (ctx) => {
  const session = await getServerAuthSession(ctx);
  
  if (!session) {
    return {
      redirect: {
        destination: "/auth",
        permanent: false,
      },
    };
  }
  
  return { props: { session } };
};
```

## Client-Side Usage

### Session Provider

```typescript
// src/pages/_app.tsx
import { SessionProvider } from "next-auth/react";

export default function App({ session, ...appProps }) {
  return (
    <SessionProvider session={session}>
      <Component {...pageProps} />
    </SessionProvider>
  );
}
```

### Using Sessions in Components

```typescript
import { useSession, signIn, signOut } from "next-auth/react";

export default function Header() {
  const { data: session, status } = useSession();

  if (status === "loading") return <Loading />;

  if (session) {
    return (
      <div>
        <p>Signed in as {session.user.email}</p>
        <img src={session.user.image} alt="Profile" />
        <button onClick={() => signOut()}>Sign out</button>
      </div>
    );
  }

  return (
    <div>
      <p>Not signed in</p>
      <button onClick={() => signIn("github")}>Sign in with GitHub</button>
    </div>
  );
}
```

## Protected Routes

### Page-Level Protection

```typescript
// Higher-order component for protection
export function withAuth<P extends {}>(Component: React.ComponentType<P>) {
  return function AuthenticatedComponent(props: P) {
    const { data: session, status } = useSession();
    const router = useRouter();

    useEffect(() => {
      if (status === "loading") return;
      if (!session) router.push("/auth");
    }, [session, status, router]);

    if (status === "loading") return <Loading />;
    if (!session) return null;

    return <Component {...props} />;
  };
}

// Usage
export default withAuth(function ProtectedPage() {
  return <div>This page requires authentication</div>;
});
```

### API Route Protection

```typescript
// tRPC protected procedure
export const protectedProcedure = t.procedure.use(({ ctx, next }) => {
  if (!ctx.session || !ctx.session.user) {
    throw new TRPCError({ code: "UNAUTHORIZED" });
  }
  return next({
    ctx: {
      session: { ...ctx.session, user: ctx.session.user },
    },
  });
});
```

## Role-Based Access Control

### User Roles

```typescript
// prisma/schema.prisma
enum Role {
  ADMIN
  ORGANIZER
  USER
}

model User {
  role Role @default(USER)
  // ... other fields
}
```

### Role Checks

```typescript
// tRPC admin procedure
export const adminProcedure = protectedProcedure.use(({ ctx, next }) => {
  if (ctx.session.user.role !== "ADMIN") {
    throw new TRPCError({ 
      code: "FORBIDDEN",
      message: "Admin access required" 
    });
  }
  return next({ ctx });
});

// Component role checking
function AdminPanel() {
  const { data: session } = useSession();
  
  if (session?.user.role !== "ADMIN") {
    return <div>Access denied</div>;
  }
  
  return <div>Admin content</div>;
}
```

## Custom Auth Pages

### Sign In Page

```typescript
// src/pages/auth/index.tsx
import { getProviders, signIn, getSession } from "next-auth/react";
import { GetServerSideProps } from "next";

export default function SignIn({ providers }) {
  return (
    <div>
      <h1>Sign in to Hackcontrol</h1>
      {Object.values(providers).map((provider) => (
        <div key={provider.name}>
          <button onClick={() => signIn(provider.id)}>
            Sign in with {provider.name}
          </button>
        </div>
      ))}
    </div>
  );
}

export const getServerSideProps: GetServerSideProps = async (context) => {
  const session = await getSession(context);
  
  if (session) {
    return { redirect: { destination: "/app" } };
  }

  const providers = await getProviders();
  return { props: { providers } };
};
```

## Database Integration

### Prisma Adapter Models

```prisma
// NextAuth required models
model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String?
  access_token      String?
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String?
  session_state     String?
  user              User    @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}
```

## GitHub OAuth Setup

### 1. Create GitHub OAuth App

1. Go to GitHub Settings > Developer settings > OAuth Apps
2. Click "New OAuth App"
3. Fill in details:
   - **Application name**: "Hackcontrol"
   - **Homepage URL**: "http://localhost:3000" (dev) or your domain
   - **Authorization callback URL**: "http://localhost:3000/api/auth/callback/github"

### 2. Environment Variables

```env
GITHUB_CLIENT_ID="your-client-id"
GITHUB_CLIENT_SECRET="your-client-secret"
```

### 3. Production Setup

For production, update:
- Homepage URL to your domain
- Callback URL to `https://yourdomain.com/api/auth/callback/github`

## Security Features

### CSRF Protection
- Built-in CSRF protection
- Automatic token validation
- Secure cookie handling

### Session Security
```typescript
// Secure session configuration
session: {
  strategy: "jwt",
  maxAge: 30 * 24 * 60 * 60, // 30 days
  updateAge: 24 * 60 * 60,    // 24 hours
},
```

### Cookie Configuration
```typescript
cookies: {
  sessionToken: {
    name: "next-auth.session-token",
    options: {
      httpOnly: true,
      sameSite: "lax",
      path: "/",
      secure: process.env.NODE_ENV === "production",
    },
  },
},
```

## Troubleshooting

### Common Issues

1. **Callback URL Mismatch**
   - Ensure GitHub OAuth app callback URL matches exactly
   - Include protocol (http/https)

2. **Environment Variables**
   - Verify GITHUB_CLIENT_ID and GITHUB_CLIENT_SECRET
   - Check NEXTAUTH_SECRET is set

3. **Database Issues**
   - Ensure Prisma schema includes NextAuth models
   - Run database migrations

### Debug Mode
```env
NEXTAUTH_DEBUG=true  # Enable detailed logging
```

## Best Practices

### Security
- Always use HTTPS in production
- Set secure environment variables
- Implement proper role-based access
- Regularly rotate secrets

### User Experience
- Provide clear sign-in/out flows
- Handle loading states gracefully
- Show appropriate error messages
- Implement session persistence

### Performance
- Use JWT for stateless sessions
- Cache user roles appropriately
- Minimize database queries in callbacks

## Next Steps

- Learn about [User Management](../features/authentication.md)
- Explore [Role-Based Features](../features/hackathon-management.md)
- Understand [Security Best Practices](../development/security.md)
