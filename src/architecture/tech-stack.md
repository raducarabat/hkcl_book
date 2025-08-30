# Technology Stack

Hackcontrol is built with a modern, production-ready technology stack focused on type safety, developer experience, and performance.

## Frontend Technologies

### Next.js 15.4.6
- **Role**: React framework and application foundation
- **Features**: SSR, SSG, API routes, file-based routing
- **Benefits**: Performance optimization, SEO, developer experience
- **Configuration**: `next.config.mjs`

### React 18.2.0
- **Role**: UI library for component-based architecture
- **Features**: Hooks, context, concurrent rendering
- **Benefits**: Component reusability, virtual DOM, large ecosystem

### TypeScript 5.1.6
- **Role**: Static type checking and enhanced developer experience
- **Features**: Type inference, interfaces, generics
- **Benefits**: Catch errors at compile time, better IDE support
- **Configuration**: `tsconfig.json`

## Styling & UI

### Tailwind CSS 3.3.3
- **Role**: Utility-first CSS framework
- **Features**: Responsive design, custom themes, JIT compilation
- **Benefits**: Rapid development, consistent design, small bundle size
- **Configuration**: `tailwind.config.cjs`

### Radix UI Primitives
- **Components**: Dialog, Tabs, Tooltip
- **Role**: Unstyled, accessible UI components
- **Benefits**: Accessibility compliance, keyboard navigation, ARIA support

### clsx 2.0.0
- **Role**: Conditional CSS class utility
- **Usage**: Dynamic styling based on state/props
- **Benefits**: Clean conditional styling patterns

### Framer Motion 10.15.0
- **Role**: Animation library for React
- **Features**: Declarative animations, gesture handling
- **Usage**: Page transitions, micro-interactions
- **Location**: `src/animations/`

## Backend & API

### tRPC 10.36.0
- **Role**: Type-safe API layer
- **Features**: End-to-end type safety, auto-generated clients
- **Benefits**: No code generation, excellent DX, runtime safety
- **Structure**: 
  - Router definitions in `src/trpc/routers/`
  - Client setup in `src/trpc/api.ts`

### Prisma 5.1.0
- **Role**: Database ORM and query builder
- **Features**: Type-safe queries, migrations, schema management
- **Benefits**: Auto-generated client, great TypeScript integration
- **Schema**: `prisma/schema.prisma`

### NextAuth.js 4.24.11
- **Role**: Authentication solution
- **Features**: Multiple providers, session management, security
- **Provider**: GitHub OAuth
- **Configuration**: `src/lib/auth.ts`

## Database

### CockroachDB
- **Role**: Distributed SQL database
- **Features**: Horizontal scaling, ACID compliance, PostgreSQL compatibility
- **Benefits**: High availability, strong consistency, cloud-native
- **Connection**: Via Prisma with PostgreSQL protocol

## Data & Validation

### Zod 3.25.76
- **Role**: Schema validation library
- **Usage**: 
  - API input validation
  - Environment variable validation
  - Form validation
- **Location**: `src/schema/` and `src/env/`

### Superjson 1.13.1
- **Role**: JSON serialization with additional types
- **Features**: Date, undefined, BigInt, Map, Set support
- **Usage**: tRPC data transformation
- **Benefits**: Preserves complex JavaScript types over network

### React Hook Form 7.45.2
- **Role**: Form state management
- **Features**: Minimal re-renders, easy validation integration
- **Benefits**: Performance, DX, Zod integration

## Development Tools

### ESLint 8.46.0
- **Role**: Code linting and style enforcement
- **Configuration**: `.eslintrc.json`
- **Extends**: Next.js recommended config

### Prettier 3.0.0
- **Role**: Code formatting
- **Features**: Opinionated formatting, Tailwind class sorting
- **Plugins**: `prettier-plugin-tailwindcss`
- **Configuration**: `prettier.config.cjs`

### TypeScript ESNext
- **Target**: ES5 for broad compatibility
- **Module**: ESNext for optimal bundling
- **Features**: Strict mode, path mapping (`@/*`)

## Additional Libraries

### UI Enhancement
- **Sonner 0.6.2**: Toast notifications
- **Canvas Confetti 1.6.0**: Celebration animations
- **@uiball/loaders 1.3.0**: Loading indicators
- **Iconoir**: SVG icon library

### Utilities
- **nanoid 5.1.5**: Unique ID generation
- **next-seo 6.1.0**: SEO optimization
- **nextjs-progressbar 0.0.16**: Page loading indicator

## Package Manager

The project supports multiple package managers:
- **npm** (default)
- **pnpm** (recommended for speed)
- **yarn** (classic)

All package managers work with the same `package.json` configuration.

## Build Pipeline

### Development
```bash
npm run dev          # Start dev server
npm run lint         # Run ESLint
npm run ts:check     # TypeScript compilation check
npm run format       # Format code with Prettier
```

### Production
```bash
npm run build        # Build for production
npm run start        # Start production server
```

### Database
```bash
npx prisma migrate dev    # Apply schema changes
npx prisma generate      # Generate Prisma client
npm run studio          # Open Prisma Studio
```

## Architecture Benefits

1. **Type Safety**: Full-stack TypeScript eliminates entire classes of runtime errors
2. **Developer Experience**: Hot reloading, auto-completion, and excellent tooling
3. **Performance**: Optimized builds, code splitting, and efficient rendering
4. **Maintainability**: Clear separation of concerns and modular architecture
5. **Scalability**: Serverless-ready components and efficient database layer

## Next Steps

- Explore [Project Structure](project-structure.md)
- Learn about [Database Schema](database-schema.md)
- Dive into [tRPC Implementation](../technologies/trpc.md)
