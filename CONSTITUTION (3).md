# Liverpool Project Engineering Constitution
**Version:** 1.0.0

This constitution is the supreme engineering authority for the Liverpool Project. All code, architecture decisions, and engineering practices must strictly adhere to the rules and standards defined within this document. No pull request may be merged if it violates these tenets.

## Mission
To deliver a high-performance, reliable, and scalable sports analytics and fan engagement platform that provides real-time insights, secure ticketing, and seamless user experiences for global audiences.

## Core Values
1. **Correctness over speed:** It is better to deliver a working, bug-free feature later than a broken feature today.
2. **Security over convenience:** Security practices are non-negotiable and must never be bypassed for developer or user convenience.
3. **Simplicity over cleverness:** Code must be readable and maintainable by any engineer on the team.
4. **Maintainability over shortcuts:** Technical debt must be deliberate, documented, and short-lived.
5. **Observability over assumptions:** If we cannot measure it, we do not know if it works.
6. **Explicitness over magic:** Avoid implicit behaviors, hidden state, and overly abstracted frameworks.
7. **Automation over manual processes:** CI/CD, testing, and infrastructure provisioning must be automated.
8. **Testing over trust:** All code must prove its correctness through automated tests.

## Technology Stack

### Required Technologies
| Category | Technology |
| :--- | :--- |
| **Frontend** | React, Next.js, Tailwind CSS |
| **Backend** | Go (Golang), Fiber |
| **Database** | PostgreSQL |
| **Caching/Queue** | Redis |
| **Languages** | TypeScript (Frontend), Go (Backend) |
| **Infrastructure** | Docker, Kubernetes, AWS |

### Forbidden Technologies & Practices
* **Plain JavaScript** in application code (TypeScript is strictly required).
* **Unmaintained dependencies** (libraries without updates in the last 12 months).
* **Experimental libraries** in production without explicit architectural approval.
* **ORMs with hidden N+1 queries** (raw SQL or query builders like `sqlc` are preferred in Go).

## Repository Structure
The project utilizes a monorepo structure to share types and configurations while keeping deployment boundaries clear.

```text
liverpool-project/
├── apps/
│   ├── web/                # Next.js frontend application
│   └── api/                # Go backend API services
├── packages/
│   ├── shared-types/       # Shared TypeScript definitions
│   ├── ui-components/      # Reusable React components
│   └── config/             # Shared ESLint, Prettier, and TS configs
├── infrastructure/         # Terraform and Kubernetes manifests
├── docs/                   # Architecture Decision Records (ADRs) and guides
├── .github/                # CI/CD workflows
└── README.md
```

## Language/Code Standards

### Naming Conventions
| Context | Convention | Example |
| :--- | :--- | :--- |
| TypeScript Variables/Functions | `camelCase` | `getUserData()` |
| TypeScript Types/Interfaces/Classes | `PascalCase` | `UserProfile` |
| React Components | `PascalCase` | `TicketWidget.tsx` |
| Go Variables/Functions | `camelCase` | `calculateStats()` |
| Go Structs/Interfaces | `PascalCase` | `MatchResult` |
| Database Tables/Columns | `snake_case` | `user_accounts`, `created_at` |
| Environment Variables | `UPPER_SNAKE_CASE` | `DATABASE_URL` |

### File and Component Size Limits
* **Target Size:** 300 lines of code per file.
* **Mandatory Refactor:** 500 lines of code per file. Any file exceeding this limit will fail CI linting and must be broken down into smaller, composable modules.

## Frontend Standards
* **State Management:** Use React Context for global UI state and React Query (TanStack Query) for server state. Avoid Redux unless strictly necessary for complex client-side state.
* **Data Fetching:** All data fetching must be typed end-to-end. Use Server Components in Next.js where possible to reduce client-side JavaScript.
* **Styling:** Use Tailwind CSS via utility classes. Avoid inline styles and custom CSS files unless implementing complex animations.
* **Component Structure:** Separate business logic from presentation. Use custom hooks to encapsulate logic.

## Backend/API & Validation Standards
* **API Design:** RESTful principles must be followed. Use standard HTTP methods and status codes.
* **Validation:** All incoming requests must be validated at the boundary. Use `validator` in Go. Never trust client input.
* **Database Access:** Use explicit SQL queries. Migrations must be version-controlled and reversible.
* **Concurrency:** Use Go routines and channels responsibly. Always pass `context.Context` down the call stack to handle timeouts and cancellations.

## Error Handling
Errors must be categorized and handled explicitly. Do not swallow errors.

| Category | HTTP Status | Description |
| :--- | :--- | :--- |
| `VALIDATION_ERROR` | 400 | Invalid input data provided by the client. |
| `AUTHENTICATION_ERROR` | 401 | Missing or invalid authentication credentials. |
| `AUTHORIZATION_ERROR` | 403 | Authenticated user lacks required permissions. |
| `BUSINESS_ERROR` | 409 / 422 | Domain rule violation (e.g., ticket already sold). |
| `EXTERNAL_SERVICE_ERROR` | 502 / 504 | Failure communicating with a third-party API. |
| `INFRASTRUCTURE_ERROR` | 500 | Database connection failure, out of memory, etc. |
| `UNKNOWN_ERROR` | 500 | Unhandled panic or unexpected system failure. |

## Logging
Logs must be structured as JSON in production.

### Required Structured Log Fields
* `event`: String identifier for the action (e.g., `user_login_attempt`).
* `timestamp`: ISO 8601 formatted UTC timestamp.
* `requestId`: Unique UUID for distributed tracing.
* `userId?`: ID of the authenticated user (if applicable).
* `metadata?`: Additional context (e.g., `ticketId`, `matchId`).

## Security
* **Authentication & Authorization:** Must occur server-side. The frontend is strictly a presentation layer and cannot be trusted to enforce security rules.
* **Secrets Handling:**
  * Must be injected via environment variables.
  * Must be managed by a secure Secret Manager (e.g., AWS Secrets Manager, HashiCorp Vault).
  * **Never commit secrets** to version control. Pre-commit hooks must scan for entropy and known secret patterns.
* **Dependency Policy:**
  * Must pass automated security scans (e.g., Dependabot, Trivy).
  * Must pass license review (no copyleft licenses like GPL in proprietary code).
  * Must be actively maintained.
  * Prefer building over adding a dependency when the required functionality is small and easily implemented.

## Accessibility
* **Standard:** Must comply with WCAG 2.1 AA standards.
* **Implementation:** Use semantic HTML elements. Ensure proper ARIA attributes are used for custom interactive components.
* **Validation:** Automated accessibility testing (e.g., axe-core) must run in CI for all frontend components.

## Performance
* **Frontend:** Must meet Google Core Web Vitals thresholds (LCP < 2.5s, FID < 100ms, CLS < 0.1).
* **Backend:** API response times must be < 200ms at the 95th percentile.
* **Database:** All queries filtering or sorting data must utilize appropriate indexes.

## Testing
* **Minimum Coverage:** 80% minimum overall coverage, 95% for critical business logic (e.g., ticketing, payments, authentication).
* **Required Test Types:**
  * **Unit Tests:** For all pure functions, utilities, and isolated components.
  * **Integration Tests:** For API endpoints, database queries, and cross-module workflows.
  * **End-to-End (E2E) Tests:** For critical user journeys (e.g., purchasing a ticket, logging in).

## CI/CD
Every Pull Request must pass the following automated gates before it can be merged:
1. **Lint:** Code style and static analysis checks.
2. **Typecheck:** TypeScript compilation and Go vet.
3. **Unit tests:** All unit tests must pass.
4. **Integration tests:** All integration tests must pass against a localized ephemeral database.

## Documentation
* **READMEs:** Every application and package must have a `README.md` detailing setup, execution, and testing instructions.
* **Architecture:** Significant architectural changes must be documented using Architecture Decision Records (ADRs) in the `docs/` directory.
* **API Documentation:** Backend APIs must expose an OpenAPI (Swagger) specification, automatically generated from code/comments.

## Observability
* **Metrics:** Expose Prometheus metrics for application health, request rates, error rates, and durations (RED metrics).
* **Tracing:** Implement OpenTelemetry distributed tracing across the frontend and backend to track requests end-to-end.
* **Dashboards:** Maintain Grafana dashboards for all critical services. Alerts must be configured for error rate spikes and latency degradation.

## AI Development Rules
* **AI-generated code policy:** AI code is UNTRUSTED. It must be thoroughly reviewed, tested, and validated by a human engineer before being merged.
* **Agent restrictions (each without human approval):**
  * May NOT deploy to production.
  * May NOT rotate credentials.
  * May NOT modify infrastructure.
  * May NOT approve pull requests.

## Prompt/MCP/RAG Standards
* **Prompts:** Must be version-controlled, documented, and tested alongside application code. Prompt changes require standard peer review.
* **MCP (Model Context Protocol):** Integrations must be least-privilege, fully auditable, and easily revocable.
* **RAG (Retrieval-Augmented Generation):** Sources must be trusted, versioned, and explicitly source-attributed in the output.

## Code Review Standards
Every Pull Request description must explicitly answer the following questions:
1. **What changed?**
2. **Why?**
3. **Risks?**
4. **Rollback plan?**
5. **Testing evidence?**

## Git Standards

### Branch Conventions
* `feature/` - New features
* `bugfix/` - Bug fixes
* `hotfix/` - Urgent production fixes
* `chore/` - Maintenance, dependencies, tooling

### Commit Conventions
Follow Conventional Commits:
* `feat:` A new feature
* `fix:` A bug fix
* `refactor:` Code change that neither fixes a bug nor adds a feature
* `test:` Adding or correcting tests
* `docs:` Documentation changes
* `perf:` A code change that improves performance
* `chore:` Changes to the build process or auxiliary tools

## Dependency Rules
* All new dependencies require justification in the PR.
* Must pass security scan.
* Must pass license review.
* Must be maintained.
* Prefer building over adding a dependency when smaller.

## Definition of Done
A task is not complete until ALL of the following are true:
- [ ] Requirements implemented.
- [ ] Tests written.
- [ ] Tests passing.
- [ ] Typecheck passing.
- [ ] Lint passing.
- [ ] Security review completed.
- [ ] Documentation updated.
- [ ] Accessibility validated.
- [ ] Performance validated.
- [ ] Code reviewed and approved.

## Non-Negotiable Rules (NEVER / ALWAYS)
* **NEVER** commit secrets, API keys, or passwords to version control.
* **NEVER** bypass CI/CD checks or force-push to `main`.
* **NEVER** deploy unreviewed AI-generated code.
* **NEVER** trust client-side data without server-side validation.
* **ALWAYS** write tests for new features and bug fixes.
* **ALWAYS** handle errors explicitly and log them with context.
* **ALWAYS** leave the codebase cleaner than you found it.

## Amendment Process
To modify this constitution, the following process must be strictly followed:
1. **Written proposal:** Submit a PR modifying this document with a detailed justification.
2. **Architecture review:** The proposal must be reviewed by the lead architects.
3. **Team approval:** Requires a majority consensus from the core engineering team.
4. **Version increment:** Upon approval, the version number at the top of this document must be incremented following Semantic Versioning.