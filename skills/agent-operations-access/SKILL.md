---
name: agent-operations-access
description: "Explores a project, related repositories, deployment architecture, and external services to produce a least-privilege .agent.example.vars template with verified credential-provisioning links."
disable-model-invocation: true
---

# Designing Agent Operations Access

Produce a root-level `.agent.example.vars` describing the minimum useful
credentials an AI operations agent needs to monitor, troubleshoot, test, and
safely remediate a project.

Include verified official links for creating each token, service account,
project, tenant, or equivalent identity.

Do not create credentials, expose secret values, or modify application runtime
secrets.

## Goals

The access design must:

- Support practical incident investigation.
- Minimize persistent privileges.
- Share credentials across environments whenever safe.
- Separate READ and WRITE capabilities.
- Separate DEV and PROD only at genuine security boundaries.
- Prefer audited workflows over direct infrastructure administration.
- Document temporary break-glass access without provisioning it persistently.
- Remain separate from application runtime credentials.
- Tell operators where and how to provision each credential.

## Naming schema

Use:

```text
AGENT_<ACCESS>_<SERVICE>
AGENT_<ACCESS>_<ENVIRONMENT>_<SERVICE>
```

Where:

- `<ACCESS>` is `READ` or `WRITE`.
- `<ENVIRONMENT>` is `DEV` or `PROD`.
- Omit the environment when one credential can safely cover both.
- `<SERVICE>` is a stable uppercase provider or control-plane name.
- Do not use `SHARED`; an omitted environment already means shared.

Examples:

```text
AGENT_READ_GITHUB
AGENT_WRITE_GITHUB
AGENT_READ_DEV_SUPABASE
AGENT_READ_PROD_SUPABASE
```

## Discovery workflow

### 1. Read repository guidance

Read applicable:

- `AGENTS.md`
- Development and testing documentation
- Deployment and operations documentation
- Compliance and security guidance
- Existing environment templates

Respect shared-development and production-safety instructions.

### 2. Inventory the local project

Inspect:

- Package manifests and lockfiles
- Environment types and templates
- Deployment and infrastructure configuration
- CI/CD and scheduled workflows
- Database and migration tooling
- Authentication clients
- Storage, queue, scheduler, and email bindings
- Logging, tracing, metrics, and alerts
- External AI, search, media, payment, and data providers
- Backup and restore workflows
- Container registries and artifact stores
- DNS, TLS, edge, and hosting configuration

Search for environment-variable **names**, SDK imports, bindings, API hosts,
project identifiers, and deployment resources. Never print secret values.

Classify each discovered dependency as:

1. Production runtime dependency
2. Operations control plane
3. CI/CD or artifact system
4. Development/test-only system
5. Deprecated or historical integration

Exclude inactive, deprecated, and test-only systems unless they participate in
an active operations workflow.

### 3. Explore related repositories

Determine whether infrastructure, deployment, observability, backend,
shared-service, or secrets-management code lives elsewhere.

Use:

- Repository remotes
- Documentation links
- Package metadata
- Workspace context
- References in CI/CD configuration
- Repository-understanding tools for remote repositories

Inspect only materially related projects.

Record which repository owns:

- Infrastructure provisioning
- Deployment workflows
- Shared services
- Monitoring and alerting
- Container images
- Database migrations and backups
- Secrets or configuration management

Do not infer services merely because they are commonly used.

## Credential-provisioning links

For every proposed credential, attempt to locate an official page for creating
the required token, service account, application, project, or tenant.

### Discovery order

1. Look for official console or documentation URLs already present in the code
   or repository documentation.
2. Inspect related repositories for provisioning documentation.
3. If no suitable link exists, use web search for the current official provider
   page.
4. Prefer the provider's own documentation or dashboard domain.
5. Verify the page with a web-page reader when accessible.

Useful searches include:

```text
<provider> create API token official
<provider> create read only service account
<provider> custom role API token official
<provider> create project tenant official
<provider> fine grained access token official
```

### Link preference

Prefer links in this order:

1. Direct token or service-account creation page
2. Direct project or tenant access-management page
3. Official setup documentation
4. Official dashboard landing page with a documented navigation path

Include both links when creating a tenant/project and creating its credential
are separate operations.

### Link safety

- Use only official provider domains.
- Do not guess URLs.
- Do not include temporary session, invitation, callback, or signed URLs.
- Do not include credentials in query parameters.
- Do not expose tenant or project identifiers unless they are already
  intentionally public and safe to commit.
- When an account-specific link would expose a sensitive identifier, use the
  generic official page and describe the navigation path.
- If a stable official link cannot be verified, state that explicitly instead
  of fabricating one.
- Provider permission names and console URLs can change; prefer current official
  documentation over stale repository comments.

### Template format

Place provisioning information immediately above the corresponding variable:

```dotenv
# Purpose: Query deployment history, logs, analytics, and resource state.
# Create token: https://official-provider.example/tokens
# Create account/project: https://official-provider.example/projects/new
# Required scope: Read-only access to the named project resources.
AGENT_READ_PROVIDER=
```

If token creation requires dashboard navigation:

```dotenv
# Create token: https://official-provider.example/dashboard
# Navigation: Organization settings → Service accounts → New service account
```

For environment-specific projects:

```dotenv
# Create DEV project: https://official-provider.example/new
# Create token: https://official-provider.example/access-tokens
AGENT_READ_DEV_PROVIDER=

# Use a separate credential issued inside the PROD project.
# Create token: https://official-provider.example/access-tokens
AGENT_READ_PROD_PROVIDER=
```

## Verify provider capabilities

Before recommending a credential, determine whether the provider supports:

- Read-only API keys or service accounts
- Resource or project scoping
- Environment separation
- Credential expiry and rotation
- Audit logs
- Budget and rate limits
- Custom roles
- Short-lived credentials
- Approval-gated workflows

If this cannot be established from repository sources, consult current official
provider documentation.

If a provider cannot issue a genuinely read-only machine credential, document
that limitation and recommend leaving the variable empty rather than reusing an
application runtime secret.

## Access classification

### READ

READ supports diagnosis without modifying shared state:

- Query logs, metrics, traces, and alert status
- Inspect deployments and configuration
- Read CI/CD runs and artifacts
- Read usage, quota, and billing state
- List storage metadata
- Inspect database health through approved views
- Read audit events
- Inspect user/session state when operationally necessary

READ credentials should normally be persistent.

Database READ access should use a dedicated observer role with:

- No ownership or role administration
- No DDL or DML
- No RLS bypass
- Access only to required schemas, tables, or masked views
- `default_transaction_read_only=on`
- A statement timeout
- A low connection limit

### WRITE

WRITE covers external side effects, including:

- Repository branches, pull requests, and workflow dispatches
- Deployments and rollbacks
- Database mutation or restoration
- DNS or infrastructure changes
- Alert silencing or rule changes
- User or session changes
- Prompt publication
- Package publication
- Incident notifications
- Synthetic AI, search, or media requests that consume quota

Persistent WRITE access should be exceptional.

Reasonable persistent WRITE capabilities are limited to:

- Creating reviewable branches or pull requests
- Dispatching approval-gated workflows
- Posting to a fixed incident channel
- Running low-budget, rate-limited synthetic checks

Broad provider administration, infrastructure mutation, and production-data
writes belong in the break-glass section.

## Sharing rules

Prefer one environment-neutral credential when:

- DEV and PROD use the same vendor account or project.
- One read-only identity can be scoped safely across both.
- Telemetry has reliable environment labels.
- Sharing does not unnecessarily expose production data.
- Audit logs still identify the agent.

Use separate DEV and PROD credentials when:

- They are separate database or identity projects.
- PROD contains materially more sensitive data.
- Provider permissions cannot scope resources safely.
- Independent revocation is operationally important.
- Sharing would cross a compliance boundary.

Do not split credentials merely for symmetry.

## Runtime-secret separation

Do not copy application runtime credentials into agent variables.

Normally exclude:

- Production application secret keys
- Database owner or service-role credentials
- Deployment tokens
- Signing private keys
- Watermarking keys
- Browser-automation runtime tokens
- Telemetry ingestion credentials

Telemetry ingestion credentials do not grant observability query access. Prefer
a Viewer service-account token.

## Output format

Create or update `.agent.example.vars` in the repository root.

The file must:

- Contain only comments and empty assignments.
- Explain each credential's purpose and minimum scope.
- Include verified official provisioning links where available.
- Describe navigation when a direct creation link is unavailable.
- Group credentials into:
  - Baseline operations access
  - Vendor diagnostics
  - Narrow operational WRITE access
  - Optional operations integrations
  - Break-glass WRITE access
- Describe compact JSON shapes when multiple fields are required.
- Explain environment-specific separation.
- Identify runtime secrets that must not be reused.
- Avoid real secrets, personal data, and sensitive tenant identifiers.

Example:

```dotenv
# Grafana Viewer service account for querying logs, metrics, traces, and alerts.
# Create service account:
# https://grafana.com/docs/grafana/latest/administration/service-accounts/
# Required role: Viewer
# Format: {"url":"https://...","token":"..."}
AGENT_READ_GRAFANA=

# Separate database observers because DEV and PROD are isolated projects.
# Create role using the project's approved database-administration workflow.
AGENT_READ_DEV_DATABASE=
AGENT_READ_PROD_DATABASE=

# Creates reviewable branches/PRs and dispatches approved workflows.
# Create fine-grained token:
# https://github.com/settings/personal-access-tokens/new
# Required repository permissions: Contents and Pull requests as needed;
# Actions only when workflow dispatch/rerun is required.
AGENT_WRITE_GITHUB=
```

Break-glass credentials should normally remain comments:

```dotenv
# Create only during an approved incident, then revoke immediately.
# AGENT_WRITE_PROD_DATABASE approved production repair or restore only
```

## Local secret safety

Ensure `.agent.vars` is ignored by the root `.gitignore`.

Do not ignore `.agent.example.vars`; the template must remain tracked.

Never write actual credentials while generating or validating the template.

## Review questions

Before finalizing, verify:

1. Does each service appear in active code or operations configuration?
2. Is each credential needed for a concrete task?
3. Can observability provide the same information?
4. Can one credential safely cover multiple environments?
5. Is each persistent WRITE capability narrow and reversible?
6. Can remediation use GitHub or another audited workflow?
7. Are production database and identity boundaries preserved?
8. Are synthetic checks budgeted and rate-limited?
9. Are runtime, signing, and ingestion secrets excluded?
10. Is break-glass access clearly distinguished?
11. Is every provisioning link official and current?
12. Are sensitive tenant identifiers absent from committed links?

Remove speculative, redundant, or unverifiable entries.

## Verification

After writing the template:

1. Run `bash -n .agent.example.vars`.
2. Verify assignment names match:

   ```regex
   ^AGENT_(READ|WRITE)_(?:(DEV|PROD)_)?[A-Z0-9][A-Z0-9_]*$
   ```

3. Verify names are unique.
4. Verify no name contains `_SHARED_`.
5. Verify provisioning URLs use official provider domains.
6. Check that URLs contain no credentials or signed parameters.
7. Run `git diff --check`.
8. Verify `.agent.vars` is ignored.
9. Review the final diff for secrets and unrelated changes.

Report:

- Services discovered
- Related repositories inspected
- Credentials shared across environments
- Credentials separated by environment and why
- Persistent WRITE capabilities retained and why
- Break-glass capabilities documented
- Provisioning links found or unavailable
- Verification results