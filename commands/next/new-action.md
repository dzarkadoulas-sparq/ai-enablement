Scaffold a Server Action. Action name: `$ARGUMENTS`.

1. **Place the file** at `features/<feature>/actions/<name>.ts` (or
   extend an existing `actions.ts` if small).

2. **Required structure**

   ```ts
   'use server';
   import 'server-only';
   import { z } from 'zod';
   import { requireUser } from '@/lib/auth';
   import { revalidateTag } from 'next/cache';
   // ... other imports

   const Input = z.object({ /* fields */ });
   type Input = z.infer<typeof Input>;

   type Result<T> =
     | { ok: true; data: T }
     | { ok: false; error: { code: string; message: string } };

   export async function <name>(formDataOrInput: FormData | Input):
     Promise<Result</* output */>> {
     const user = await requireUser();          // AuthN
     // AuthZ (if scoped): requireRole('admin') or ownership check
     const parsed = Input.safeParse(/* normalize from FormData */);
     if (!parsed.success) {
       return { ok: false, error: { code: 'VALIDATION', message: '...' } };
     }
     try {
       const data = await /* do the work */;
       revalidateTag(/* specific tag */);
       return { ok: true, data };
     } catch (e) {
       // Log via lib/reporting.ts with correlation id; return safe error.
       return { ok: false, error: { code: 'INTERNAL', message: 'Something went wrong' } };
     }
   }
   ```

3. **Mandatory constraints**
   - First import: `import 'server-only';`.
   - First statement in the body: an auth check (`requireUser` /
     `requireRole`).
   - Zod validation of every input field. No `any`.
   - No raw DB errors or stack traces in the returned `error.message`.
   - Specific `revalidateTag` / `revalidatePath` — never the bare root.
   - Rate-limit sensitive actions (auth, billing, delete) via the limiter
     in `lib/ratelimit.ts`.

4. **Hook it up in the calling component**
   - Server Component: `<form action={<name>}>` with `useFormState` /
     `useFormStatus` for the pending + error UI.
   - Client Component (if needed): `const { execute, pending } = useAction(<name>)`.

5. **Test**
   - Unit test: `features/<feature>/__tests__/<name>.test.ts` calling the
     action directly with an in-memory or containerized DB. Cover happy
     path, validation failure, auth failure.

6. **Verify**
   - `pnpm lint --fix && pnpm typecheck && pnpm test`.
   - Playwright MCP: exercise the form in the browser. Confirm the
     network tab shows the action POST and the page revalidates.

7. **Summary** — file path, inputs, tags revalidated, auth requirement,
   test coverage.

Respect CLAUDE.md rules: never expose DB client to the browser; never
accept unvalidated input; never catch-and-swallow DB errors silently.
