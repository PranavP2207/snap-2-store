# Decisions: Snap-2-Store

Last updated: 2026-10-17

## Active decisions

### D-001: Import the existing repository and limit this task to review

- Date: 2026-10-17
- Status: superseded
- Source: User request to import `Gautamx025/project2.0` and "review it for now".
- Decision: Clone the default branch directly into the empty Hackathon workspace, preserve its origin and history, and review the full initial commit without application changes.
- Reason: This makes the requested repository the actual project while preserving a clear baseline.
- Alternatives: A nested clone would add an unnecessary app root; implementation changes would exceed the current request.
- Consequences: Review findings remain open. Required context records are local additions, with no commit or push requested.

## Decision history

No superseded decisions recorded. Existing technology choices are observed implementation facts, not newly approved architecture decisions.

### D-002: Implement the complete local full-stack app

- Date: 2026-10-17
- Status: accepted
- Source: New user build brief; supersedes D-001's review-only scope.
- Decision: Use this imported repository, replace its three-file mock, implement the exact requested Next.js/FastAPI stack, and commit M1â€“M6 after verification. Local delivery with Docker/manual commands; no hosted Sites deployment is requested.
- Reason: User requests a working local full project and milestone commits.
- Alternatives: Keeping the old static demo cannot meet the brief.
- Consequences: Original source remains recoverable in Git at 70eab0c; context files remain canonical at root.

### D-003: Local-first, clearly labeled fallback and durable shared state

- Date: 2026-10-17
- Status: accepted
- Source: Agent defaults within the user's autonomous build scope.
- Decision: Persist jobs/listings/events as per-record JSON under ignored data/, with atomic writes and Redis queue coordination; use one stateless API process with BackgroundTasks locally, RQ workers and shared data volume in Compose. Production can replace the repository interface with a shared database/object store. Demo uses local Pillow compositing, deterministic sample copy and labeled example prices; no network required.
- Reason: Offline demos and restart-safe history without adding an unrequested database service.
- Alternatives: In-memory records lose history; SQLite adds locking concerns across worker processes.
- Consequences: Local/Compose shared filesystem is required; multi-host deployment needs shared durable storage and authentication before public exposure.

### D-004: Retired model compatibility

- Date: 2026-10-17
- Status: accepted
- Source: Official provider documentation reviewed during implementation (docs/provider-reference.md).
- Decision: Keep Gemini as primary with an explicit GEMINI_MODEL env override; use the currently documented Flash model because Gemini 1.5 Flash is retired. Keep GPT-4o selectable. Use documented fal SAM2 point prompts and FLUX Kontext editing; support Replicate via configurable model/version.
- Reason: Hard-coding a retired endpoint would make real-key mode unusable.
- Alternatives: Preserve the requested 1.5 model string and always fall back; rejected because real provider behavior matters.
- Consequences: Provider/model availability varies by account; failures are visible, timed, retried and labeled as fallback.

### D-005: Bundled sample licensing and draft approval semantics

- Date: 2026-10-17
- Status: accepted
- Source: User permits sourced license-free placeholders and requires approval before store writes; agent implementation choices.
- Decision: Bundle three free Unsplash License photos with source/photographer attribution in docs/sample-provenance.md. They are not public domain. The six-stage pipeline ends at a timed draft-assembly stage; only a later explicit approval performs timed draft-store sync. No automatic stage can write to a store.
- Reason: Offline input assets and a clear merchant approval boundary.
- Alternatives: Runtime hotlinks break offline demos; publishing in the automatic pipeline violates the approval requirement.
- Consequences: Sample metadata/prices remain illustrative and visibly labeled; processed images are local demo composites when providers are unavailable.

### D-006: Precomputed offline masks and isolated final QA

- Date: 2026-10-17
- Status: accepted
- Source: Agent implementation choices within the user's requirement for realistic offline results and autonomous verification.
- Decision: Bundle rembg/u2netp-derived alpha cutouts for the three licensed sample photos; recognize exact sample uploads by content hash. General demo uploads retain the explicit heuristic fallback. Include CPU rembg for live local fallback, warming its model separately. Force Playwright demo mode with separate data and environment files and use production-server QA for final screenshots.
- Reason: Improves sample foreground quality without runtime Internet, avoids pretending fixture recognition is vision, and prevents tests from touching saved credentials or live stores.
- Alternatives: Runtime model downloads would break an offline stage demo; approximate masks on all samples visibly retained backgrounds.
- Consequences: Precomputed results are always labeled demo; live-provider account verification remains separate. Model weights stay ignored under data/. Original photo and derivative provenance is in docs/sample-provenance.md.

### D-007: Deliver essential project files without generated QA history

- Date: 2026-10-17
- Status: accepted
- Source: User explicitly requests pushing to the original GitHub repository as a collaborator, with important files only; exact file selection is an implementation choice.
- Decision: Push source, runtime sample assets, tests, dependency locks, environment templates, Docker configuration and essential documentation, including the four canonical context files. Exclude the pasted build brief, generated Next.js agent notes, screenshots and visual QA notes from every outgoing commit. Keep these files locally and preserve the original milestone history on a local backup branch.
- Reason: A clean clone should have everything needed to run, test and maintain the app without bulky generated outputs or personal session material.
- Alternatives: Deleting generated files only in a final commit would still upload them in earlier Git history. Squashing everything would discard the requested milestone sequence.
- Consequences: The unpublished milestone commits receive new hashes while retaining their sequence, messages and authorship. The existing remote baseline is preserved and delivery uses a normal fast-forward push, never force push. No runtime implementation changes are needed.

### D-008: Follow Vercel's global agent setup before project deployment

- Date: 2026-10-17
- Status: accepted
- Source: User explicitly asks to fetch and follow `https://vercel.com/get-started.md`.
- Decision: Install the Vercel CLI globally and the official plugin for Codex only in user scope; configure the shared `https://mcp.vercel.com` server in user-wide Codex settings, authenticate through the browser and retain confirmation for MCP writes. Do not install duplicate standalone skills or create/link a project during global onboarding.
- Reason: The requested guide distinguishes account/agent preparation from project-specific deployment and requires separate CLI and MCP authentication.
- Alternatives: A project-scoped installation would need repetition across projects; creating a Vercel project now would skip the guide's explicit onboarding boundary.
- Consequences: Credentials and account configuration stay outside this repository. Codex may need a reload to discover newly installed commands/tools. Deploying the complete current app still requires durable backend storage and job execution, while the existing frontend already supports an external backend through `BACKEND_URL`.

### D-009: Deploy the complete demo on Vercel with durable cloud state

- Date: 2026-10-17
- Status: accepted
- Source: User explicitly requests deployment on Vercel; platform implementation choices are within that scope.
- Decision: Use Vercel Services for the existing Next.js/FastAPI pair, free managed Upstash Redis for cloud jobs/distributed locks, and private Blob storage for immutable media. Disable automatic paid database upgrades. Keep the local filesystem/Redis/manual/Docker paths available. Begin with labeled demo providers, isolate cloud visitor histories using signed browser workspaces, and manage cloud credentials through deployment environment variables instead of a writable public settings file.
- Reason: A frontend-only deploy would leave uploads/processing/history broken. Vercel's ephemeral filesystem cannot provide the current persistence contract.
- Alternatives: Hosting FastAPI on another provider would add an external account dependency; temporary-only files would lose listings across cloud instances.
- Consequences: Cloud runtime dependencies must fit Vercel's function bundle and 300-second execution limit. The Python Blob SDK cannot provide cache-bypassed mutable reads, so Redis holds mutable records while Blob media uses unique immutable names, including on retries. Verify the deployed workflow, persistence across clients/instances and private media access. Large local rembg model dependencies remain in the local install; cloud missing-key paths retain explicit demo fallbacks. Browser uploads are reduced when needed to keep up to three photos below the platform request limit.

### D-010: Move the canonical GitHub destination to the user's repository

- Date: 2026-10-17
- Status: accepted
- Source: User explicitly selects `https://github.com/PranavP2207/project2.0` as the new repository and says Vercel is linked to it.
- Decision: Update local `origin` to the new repository and push essential deployment changes to `main` using a normal fast-forward. Preserve the original import/delivery history as historical evidence.
- Reason: The new repository is the user's selected source for continued development and Vercel deployment. Its `main` shares the existing `eaf67df` baseline, so history does not need rewriting.
- Consequences: Future pushes target `PranavP2207/project2.0`. Vercel project metadata independently confirms this repository and production branch `main`; push `4053995` triggered a successful production build and the public alias moved to it. Generated artifacts and credentials remain excluded under D-007.

### D-011: Add an app-named repository without moving the deployment source

- Date: 2026-10-17
- Status: accepted
- Source: User asks to create a new repository on their profile with the app name and push the app there as well. Repository visibility and local remote naming are implementation choices.
- Decision: Create private `PranavP2207/snap-2-store` and add the `snap-2-store` remote for explicit `main` pushes. Retain `origin` at `PranavP2207/project2.0` and its existing Vercel connection.
- Reason: This adds the requested app-named destination while preserving the established deployment source. Private visibility avoids introducing public access without an explicit request.
- Alternatives: Replacing `origin` or relinking Vercel would turn an additional delivery into a deployment migration. Pushing every local branch would include the intentionally local historical backup.
- Consequences: Both repositories can receive the same `main` commit through explicit pushes. They do not automatically synchronize. Credentials, generated artifacts and the local backup branch remain excluded.

### D-012: Publish a curated screenshot gallery with community documentation

- Date: 2026-10-17
- Status: accepted
- Source: User explicitly requests README screenshots and a community-friendly README.
- Decision: Include four existing, visually inspected local-demo screenshots in `docs/images/` and link them with expandable previews. Organize the README around onboarding, configuration, verification and contribution guidance; retain detailed store setup and verification limits.
- Reason: New readers can see the working app and find setup/contribution instructions without confusing demo captures with live provider output.
- Alternatives: Linking ignored local files would break on GitHub; committing every QA capture would add unrelated screenshots, including account consent pages. Selecting a license or inventing CI status is outside this documentation request.
- Consequences: This explicitly extends D-007's screenshot exclusion for the four curated README assets only. Raw QA captures remain ignored, and `docs/images` is excluded from Vercel uploads. Both repositories receive the documentation update; application code remains unchanged.
