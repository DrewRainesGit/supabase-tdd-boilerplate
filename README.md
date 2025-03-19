# OctoSpark - Modern SaaS Boilerplate

A powerful, type-safe SaaS boilerplate built with Next.js 15, React, TypeScript, and Supabase. This monorepo is structured using Turborepo and PNPM for optimal development experience and performance.

Useful commands:

- `git ls-files | xargs wc -l` - This is particularly useful when you want to get a quick overview of the size of your codebase in terms of lines of code.
- `npx claude` - This is useful for Claude code as a CLI code generation tool.

## Feature Context for Octospark

### Feature 1: Social Media Comment Management System

This feature creates a unified inbox for managing comments across multiple social platforms with AI assistance. It's designed for social media managers, agencies, and brands who need to efficiently handle high volumes of engagement.

**Key components:**

- Social account integration (Meta, TikTok, etc.)
- Unified comment inbox with filtering capabilities
- AI-powered comment categorization and response suggestions
- Status management workflow (open, replied, resolved)
- Linear-style keyboard shortcuts for power users
- Command palette for advanced actions

**Primary value:** Daily operational efficiency for social media managers, allowing them to handle more accounts with less time while maintaining quality engagement.

### Feature 2: Content Strategy & Analytics

This feature analyzes social media content performance and provides strategic recommendations to improve engagement and reach.

**Key components:**

- Content performance analytics across platforms
- AI-generated content recommendations based on historical data
- Competitor analysis capabilities
- Content calendar planning based on insights
- Trend identification and opportunity alerts

**Primary value:** Strategic insights for pitching clients, planning content, and demonstrating ROI. This feature helps social media professionals make data-driven decisions and justify their strategies to clients/stakeholders.

### Integration Points

- Comments provide real-time feedback on content strategy effectiveness
- Content insights inform better comment responses
- Combined analytics show correlation between content types and engagement quality
- Unified workflow from strategy to execution to measurement

This dual-feature approach creates a complete workflow for social media professionals: strategic planning, daily execution, and performance measurement in one platform with exceptional UI/UX focused on productivity.

## Inspiration:

- https://github.com/alan2207/bulletproof-react
- https://usebasejump.com/blog/testing-on-supabase-with-pgtap#testing-authenticated
- https://github.com/dougwithseismic/turbo-2025/ (Doug is really great and knows his stuff).

## 🏗️ Project Structure

```bash
.
├── apps/
│   ├── web/                 # Next.js 15 frontend application
│   │   ├── app/            # App router pages and layouts
│   │   ├── components/     # Shared UI components
│   │   ├── features/       # Feature-specific code
│   │   ├── integration/    # Integration tests
│   │   ├── e2e/           # End-to-end tests
│   │   └── __tests__/     # Unit tests
│   └── api/                # Backend API with Hono.dev
│       ├── src/            # Source code
│       │   ├── routes/     # API routes with OpenAPI/Swagger docs
│       │   ├── middleware/ # Hono middleware
│       │   └── test/       # API tests
│       └── swagger/        # Auto-generated OpenAPI documentation
├── packages/
│   ├── supabase/          # Supabase configuration and migrations
│   ├── typescript-config/  # Shared TypeScript configurations
│   └── eslint-config/     # Shared ESLint configurations
```

## 🚀 Getting Started

### Prerequisites

- Node.js 18+ and PNPM
- Docker and Docker Compose
- Supabase CLI

### Initial Setup

1. Clone the repository and install dependencies:

```bash
git clone <repository-url>
cd OctoSpark
pnpm install
```

2. Set up environment variables:

```bash
cp apps/web/.env.example apps/web/.env.local
cp apps/api/.env.example apps/api/.env
```

3. Start the Supabase local stack:

```bash
cd packages/supabase
supabase start
```

4. Start the development servers:

```bash
pnpm dev
```

### API Documentation

The API is built with [Hono](https://hono.dev), a modern, lightweight, and type-safe web framework. We use OpenAPI/Swagger for API documentation with automatic type generation from Zod schemas.

To access the API documentation:

1. Start the API server in development mode:

```bash
cd apps/api
pnpm dev
```

2. Access the documentation:

- OpenAPI JSON: http://localhost:8080/api-docs
- Swagger UI: http://localhost:8080/swagger

The API documentation is automatically generated from our route definitions and Zod schemas, ensuring it's always up-to-date with the actual implementation.

Example of a documented API route:

```typescript
import { OpenAPIHono } from "@hono/zod-openapi";
import { z } from "@hono/zod-openapi";

const userSchema = z
  .object({
    id: z.string().openapi({
      example: "123",
    }),
    name: z.string().openapi({
      example: "John Doe",
    }),
  })
  .openapi("User");

const route = {
  method: "get",
  path: "/users/{id}",
  responses: {
    200: {
      content: {
        "application/json": {
          schema: userSchema,
        },
      },
      description: "User details",
    },
  },
};

app.openapi(route, (c) => {
  // Implementation
});
```

## 🧪 Testing Strategy

We follow a comprehensive testing approach with a 60/40 split between integration and unit tests.

## Testing RLS Policies

We use a factory-based approach for testing RLS policies and authenticated endpoints. Each test user gets a JWT token that can be used to create authenticated clients:

```typescript
import { describe, it, expect } from "vitest";
import { createTestUser, createTestOrganization } from "../factories";
import { createAuthenticatedClient } from "../test-utils";

describe("Organization RLS Policies", () => {
  it("should enforce member-only access", async () => {
    // 1. Create test users with tokens
    const [adminUser, memberUser, nonMemberUser] = await Promise.all([
      createTestUser(),
      createTestUser(),
      createTestUser(),
    ]);

    // 2. Create an organization with admin
    const { organization } = await createTestOrganization({
      userId: adminUser.user.id,
    });

    // 3. Test access with different user roles
    const adminClient = createAuthenticatedClient(adminUser.token);
    const memberClient = createAuthenticatedClient(memberUser.token);
    const nonMemberClient = createAuthenticatedClient(nonMemberUser.token);

    // Admin should see all members
    const { data: adminData } = await adminClient
      .from("organization_members")
      .select("*")
      .eq("organization_id", organization.id);
    expect(adminData).toHaveLength(1);

    // Non-member should see nothing
    const { data: nonMemberData } = await nonMemberClient
      .from("organization_members")
      .select("*")
      .eq("organization_id", organization.id);
    expect(nonMemberData).toHaveLength(0);
  });
});
```

### Key Testing Patterns

1. **User Factory Functions**:

   ```typescript
   // Create a test user with token
   const { user, token } = await createTestUser({
     email: "test@example.com", // Optional
   });
   ```

2. **Authenticated Clients**:

   ```typescript
   // Create client for RLS testing
   const client = createAuthenticatedClient(token);
   ```

3. **Testing Different Roles**:

   ```typescript
   // Test the same operation with different user roles
   const adminClient = createAuthenticatedClient(adminToken);
   const memberClient = createAuthenticatedClient(memberToken);
   const nonMemberClient = createAuthenticatedClient(nonMemberToken);
   ```

4. **Testing Server Components**:

   ```typescript
   import * as supabaseServer from "@/utils/supabase/server";
   import { vi } from "vitest";

   vi.mock("@/utils/supabase/server", () => ({
     createClient: vi.fn(),
   }));

   it("should handle server operations", async () => {
     const { token } = await createTestUser();
     const client = createAuthenticatedClient(token);

     // Mock the server client to use our authenticated client
     vi.mocked(supabaseServer.createClient).mockResolvedValue(client);
   });
   ```

### Best Practices

1. **Test Data Setup**:

   - Create test users with `createTestUser()`
   - Each user gets a unique JWT token
   - Use tokens with `createAuthenticatedClient()`

2. **RLS Testing**:

   - Test each operation with different user roles
   - Verify both successful and denied operations
   - Check error messages for proper authorization failures

3. **Token Management**:

   - Tokens are automatically generated with test users
   - Tokens are valid for the test duration
   - Never reuse tokens between tests

4. **Client Usage**:
   - Use `createAdminClient()` only for test setup
   - Use `createAuthenticatedClient()` for actual tests
   - Test both positive and negative cases

## 🔒 Supabase Integration

We use three Supabase clients for different contexts:

- `supabaseAdminClient` - Full access for administrative tasks
- `supabaseClient` - Browser-side client
- `supabaseServerClient` - Server-side operations

Located in `apps/web/utils/supabase/`.

## 🛠️ Development Workflow

1. **Type Safety First**

   ```bash
   # Check types across the monorepo
   pnpm type-check
   ```

2. **Database Migrations**

   ```bash
   # Generate new migration
   cd packages/supabase
   supabase migration new my_migration_name

   # Apply migrations
   supabase db reset
   ```

3. **Testing Changes**

   ```bash
   # Run all tests
   pnpm test:all

   # Run specific test suites
   pnpm test:unit
   pnpm test:integration
   pnpm test:e2e
   ```

## 🔒 Security Best Practices

1. **Environment Variables**:

   - Never commit `.env` files
   - Use `.env.example` as templates
   - Keep different env files for different environments:
     - `.env.local` - Local development
     - `.env.test` - Testing environment
     - `.env.production` - Production environment

2. **Supabase Security**:

   - Local development uses default development keys
   - Never share or commit service role keys
   - Use RLS policies for data access
   - Keep JWT secrets secure

3. **Testing**:
   - Use separate test database
   - Never use production credentials in tests
   - Reset test data between runs
   - Use ephemeral test tokens

## 📚 Documentation

- **PRDS** - Project Progress and Documentation within the `_mini_prds` folder

## 🔍 Key Features

- Next.js 15 App Router
- Type-safe database operations
- Comprehensive testing setup
- Local development with Supabase
- Monorepo structure with Turborepo
- Modern UI with Tailwind CSS
- SEO optimization
- Analytics integration
- Error tracking with Sentry
