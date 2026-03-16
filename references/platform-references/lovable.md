# Lovable — Platform Reference

Last updated: 2026-03-15
Docs: https://docs.lovable.dev (index at https://docs.lovable.dev/llms.txt)

## What Lovable Is

Lovable is an AI-powered app builder that generates full-stack React applications from natural language prompts. It produces React + Vite + TypeScript + Tailwind CSS + shadcn/ui frontends, with Supabase (PostgreSQL, Auth, Storage, Edge Functions) as the backend. GitHub two-way sync is supported for version control.

Lovable has two modes: **Agent mode** (implements and verifies changes end-to-end) and **Plan mode** (brainstorms and structures plans before writing code).

## Hard Constraints

| Constraint | Limit | Notes |
|-----------|-------|-------|
| **Workspace knowledge** | 10,000 characters | Shared rules across all projects in a workspace |
| **Project knowledge** | 10,000 characters | Project-specific context and instructions |
| **Design system** | Enterprise only | Requires Enterprise plan |
| **GitHub repo** | 1 per project | Cannot change linked GitHub account once connected |
| **GitHub sync** | Default branch only | Branch switching is experimental (Labs) |
| **File uploads (Supabase free)** | 50 MB per file | Storage tier-dependent |
| **Browser testing** | No canvas, file upload/download, right-click, drag-and-drop | Runs in secure virtual environment, not local browser |
| **Design system `.lovable` folder** | ~500 lines for `system.md` | Owned by design system project; cannot be edited from connected projects |

## Prompting Best Practices

These are the core principles for getting high-quality output from Lovable. Apply them in order.

### Phase 1: Lay the Foundation

**1. Plan before you prompt.**
Define what you're building before touching Lovable. Answer four questions: What is this? Who is it for? Why will they use it? What is the one key action? Vague ideas produce vague outputs.

```text
Build a one-page site for a budgeting app targeted at Gen Z freelancers.
The main CTA should be "Start Saving Smarter." Focus on a bold, expressive
aesthetic with large text and punchy colors.
```

**2. Map the user journey visually.**
Think in transitions: What does the user see first → What builds trust → What gives confidence to act → Where does the action lead. Even a simple Hero → Features → CTA sketch makes prompts 10x more effective.

**3. Get the design right first.**
Visual language is a foundation, not a polish layer. Choose a direction (calm/elegant, bold/disruptive, premium/sleek) and feed style directly into prompts using buzzwords and tone descriptors. Do not fix design problems later.

```text
Use a calm, wellness-inspired design. Soft gradients, muted earth tones,
round corners, and generous padding. Font is "Inter". Overall tone should
feel gentle and reassuring.
```

### Phase 2: Think in Systems

**4. Prompt by component, not by page.**
Build UI in modular parts. A full-page prompt gets noise; a section-based prompt gets signal. Build one component, review, refine, then move to the next. Components can be reused across pages.

```text
Create a floating menu bar with glassmorphism effect. Include Home, Search,
Music, Favorites, Add, Profile, and Settings icons. Add gentle floating
animation and smooth hover interactions.
```

**5. Design with real content.**
Never use "lorem ipsum" or "feature 1 / feature 2." The model responds to structure and intent — real words reveal layout and copy issues early that placeholder text hides.

```text
Hero section with headline: "Design Calmly." Subtext: "Turn stress into
structure with Lovable." CTA: "Start Building Free." Use copy-centered
layout with generous vertical spacing.
```

**6. Speak atomic: buttons, cards, modals.**
The smaller and more specific your UI language, the better Lovable performs. Say "a modal with a success toast after submit" not "a user interface." Layer complexity gradually: card → badge → hover states.

```text
Create a card with a user profile picture, name, and a follow button.
Add a badge for verified users, and show a tooltip when hovering over the badge.
```

**7. Use buzzwords to dial in aesthetic.**
Lovable understands terms like "minimal," "expressive," "cinematic," "playful," "premium," "developer-focused." These influence typography, spacing, shadow, border radius, and color palette. Include them early in every section prompt.

```text
Design a landing page hero that feels premium and cinematic. Use layered
depth, translucent surfaces, soft motion blur, and dramatic contrast
between headline and background.
```

### Phase 3: Build with Precision

**8. Use prompt patterns for layouts.**
Build a personal prompt library of structured, repeatable layout recipes. Most patterns follow "header → content → action." Reuse and remix across pages.

```text
Create a feature section with a centered headline, followed by three
horizontally aligned cards. Each card includes an icon on top, a headline,
and a short description. Cards should have soft shadows and lift on hover.
```

**9. Add visuals via URL.**
Drop in product demos, generated clips, or tutorial videos using URLs. Prompt for placement, style (rounded corners, autoplay, muted), and context.

**10. Layer context with the Edit button.**
Use the Edit function to focus on specific elements instead of rewriting full prompts. Be precise: say "replace," "update," or "adjust" instead of "make this better."

```text
Change the CTA button text to "Get Started" and increase the padding to
24px horizontal. Keep the existing background color and font.
```

### Phase 4: Iterate and Ship

**11. Build with backend in mind.**
Anticipate auth logic (logged in vs. logged out states), dynamic content (data from tables), and states (empty, loading, error). Shape UI as if the backend is already there.

```text
If the user is logged in via Supabase, show their profile image and name
in the top right. If not, display a "Log In" button and route them to
the auth screen.
```

**12. Version control is your friend.**
Think in milestones (layout locked, content added, logic wired). Duplicate before major changes. Include notes in prompts about what changed and why.

### Key Prompting Technique: Ask Clarifying Questions

After stating a feature request, append:
> "Ask me any questions you need in order to fully understand what I want from this feature and how I envision it."

Use Plan mode for this approach. Lovable will ask focused follow-up questions that prevent misunderstandings.

## Knowledge System

Lovable uses a two-tier knowledge system for persistent instructions.

### Workspace Knowledge (cross-project)

Best for rules that apply to ALL projects:
- Coding style (TypeScript strict mode, no `any`, prefer `const`)
- Naming conventions (camelCase functions, PascalCase components, kebab-case files)
- Preferred libraries (shadcn/ui, React Query, Zustand)
- Architecture rules (service layer for API calls, no direct `fetch` in components)
- Testing requirements (unit tests for utils/hooks, run tests after changes)
- Brand voice and UI copy rules
- Things Lovable should avoid

Only workspace owners and admins can manage workspace knowledge.

### Project Knowledge (per-project)

Best for project-specific context:
- What the application does and who it's for
- Database schema and key tables
- Architecture decisions
- Domain terminology
- Design guidelines (colors, typography, layout)
- Security or compliance requirements
- Links to API docs or external references

Anyone with edit permission can update project knowledge.

### Knowledge Best Practices

| Do | Don't |
|----|-------|
| Be specific: "Always enable TypeScript strict mode. Never use `any`." | Be vague: "Write clean code." |
| Write it like onboarding docs for a new dev | Assume context |
| Use bullet lists and direct rules | Write long paragraphs |
| Review and update when stack changes | Let knowledge go stale |
| Put shared rules in workspace, specifics in project | Duplicate rules across both levels |

### Knowledge Precedence

When workspace and project knowledge conflict, **project knowledge wins** (more specific). Instruction files in the repo (`AGENTS.md`, `CLAUDE.md`) are also read — root-level `AGENTS.md` is always read regardless of session length.

## Testing & Verification

### Browser Testing
- Runs in a secure virtual environment (not local browser)
- Can navigate, click, fill forms, capture screenshots, inspect console/network
- Agent shares your auth session
- **Cannot do:** canvas interactions, file upload/download, right-click, drag-and-drop, clipboard, subtle visual assessment
- **Safety rule:** Never ask Lovable to make a large change AND test it in the same prompt — stuck tests may cause work loss. Separate building from testing.

### Frontend Tests
- Stack: Vitest + React Testing Library + jsdom
- Test files live as `.test.tsx` next to components
- Run only when requested
- Best for: regressions, conditional rendering, complex state (forms, tables, filters)

### Edge Function Tests
- Stack: Deno built-in test runner with native TypeScript
- Run only when requested
- Best for: business rules, permissions, regression protection after changes

### Verification Workflow

For backend issues, follow this sequence:
1. Call edge function directly to reproduce with specific input
2. Apply the fix
3. Call again with same input to confirm
4. Add edge test for regression protection

## Supabase Backend

| Component | Details |
|-----------|---------|
| **Database** | Hosted PostgreSQL. Lovable generates SQL schema — you execute in Supabase SQL Editor. |
| **Auth** | Email/password + OAuth (Google, GitHub, Twitter). Disable email confirmation during dev. |
| **Storage** | File uploads up to 50 MB (free tier). Organized in buckets with generated URLs. |
| **Edge Functions** | TypeScript/Deno serverless functions for AI APIs, email, payments, scheduled tasks. |
| **RLS** | Row Level Security MUST be configured before production. Dev defaults are permissive. |
| **Secrets** | Store API keys via Supabase Edge Function secret manager — never hardcode. |

## GitHub Integration

| Behavior | Details |
|----------|---------|
| **Sync direction** | Two-way: Lovable ↔ GitHub. GitHub is source of truth once connected. |
| **Branch sync** | Default branch only. Experimental branch switching available in Labs. |
| **Repo limit** | One repo per project. Cannot change linked account after connection. |
| **Repo stability** | Renaming, moving, or deleting the repo breaks the connection permanently. |
| **Self-hosting** | Clone from GitHub and deploy to Vercel, Netlify, or custom infrastructure. |

## Design Systems (Enterprise)

- Centralized React component libraries + styling guidelines in a `.lovable` folder
- Contains `system.md` (core guidelines, ~500 lines max) and `rules/` subdirectory
- Updates propagate automatically to all connected projects
- Projects can connect to multiple design systems; priority = connection order
- `.lovable` folder is read-only from connected projects — edits only in the design system project
- May conflict with default shadcn/ui + Tailwind — remove defaults if using alternative styling

## Anti-Patterns (Don't Do These)

| Anti-Pattern | Why It Fails | Do This Instead |
|-------------|-------------|-----------------|
| Full-page prompts | Too much context; noisy, unusable output | Prompt by component/section |
| Placeholder text ("lorem ipsum") | Model can't infer layout intent | Use real or realistic copy |
| Fixing design after functionality | Requires reworking structure | Decide visual direction first |
| Vague style instructions ("make it look good") | No actionable parameters | Use specific buzzwords: "minimal," "cinematic," "premium" |
| Building + testing in one prompt | Stuck tests may lose work | Separate build prompts from test prompts |
| Skipping RLS before production | Permissive defaults expose all data | Configure Row Level Security in Supabase before launch |
| Hardcoding API keys in frontend | Client-side code is public | Use Supabase Edge Function secrets |
| Long paragraphs in knowledge | Ignored in long conversations | Use bullet lists and direct rules |
| Renaming/moving linked GitHub repo | Permanently breaks sync connection | Treat repo as immutable once linked |

## Security Checklist

- [ ] Row Level Security (RLS) enabled on all Supabase tables before production
- [ ] API keys stored in Supabase Edge Function secret manager (not hardcoded)
- [ ] Email confirmation enabled in production auth settings
- [ ] Review Workspace Security Center for dependency vulnerabilities
- [ ] Audit logs enabled for membership, project actions, and auth events
- [ ] Data privacy settings reviewed (control whether data trains AI models)

## Community Resources

- Lovable Docs: https://docs.lovable.dev
- Docs Index (for agents): https://docs.lovable.dev/llms.txt
- Supabase Docs: https://supabase.com/docs
- shadcn/ui Components: https://ui.shadcn.com
