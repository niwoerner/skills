---
name: agent-operations-access
description: "Explore a project and its operations architecture, create a compact least-privilege .agent.example.vars file, and give the operator concise setup instructions with verified provisioning links and headless-friendly agent interfaces."
---

# Agent Operations Access

Create or update `.agent.example.vars` in the repository root. Do not stop at
recommendations: inspect the project, choose the minimum useful access, and
write the file before responding.

Never create credentials, expose secret values, or reuse application runtime
secrets. The operator provisions credentials after the file is created.

## Workflow

### 1. Discover the operational path

Read applicable repository guidance, deployment configuration, CI workflows,
observability configuration, database tooling, environment templates, and
active external-service integrations. Search for names and configuration, not
secret values.

Inspect existing operations scripts, CLIs, API clients, and MCP configuration.

Inspect related repositories only when they materially own infrastructure,
deployment, observability, migrations, backups, or shared services. Exclude
inactive, deprecated, speculative, and test-only integrations.

### 2. Select access by return on investment

Include a credential in Core only when it materially shortens diagnosis or safe
remediation during the first 30 minutes of a likely incident.

Prioritize:

1. Source, CI, deployment, and artifact visibility
2. Logs, metrics, traces, and alerts
3. Read-only database investigation
4. Narrow remediation through reviewable or approval-gated workflows

Keep Core to about six credentials. Put less frequent but concrete workflows in
Optional. Omit credentials that duplicate Core access or cannot be provisioned
as suitable machine credentials.

Document broad infrastructure or production-data mutation only as commented
break-glass access. Never provision it persistently.

### 3. Verify provisioning guidance

For each included credential, find a current official provisioning page when
available. Prefer a direct token or service-account page, then official setup
documentation, then an official dashboard with a short navigation path.

- Use only official provider domains.
- Do not guess URLs or include signed, account-specific, callback, or invitation
  URLs.
- Put provisioning links in the final response, not in
  `.agent.example.vars`.
- If no stable official link is found, say so briefly instead of fabricating
  one.
- Verify the minimum role or permissions before instructing the operator.

For GitHub, prefer separate fine-grained personal access tokens (PATs) for READ
and WRITE access. Scope each PAT to the required repositories and permissions.
Use a classic PAT only when a required operation is unsupported by fine-grained
PATs, and state that reason in the response.

### 4. Select an agent-friendly interface

For each Core or Optional service, choose one primary interface and any
materially useful fallback. Prefer, in order:

1. Existing audited project scripts or wrappers
2. An official non-interactive CLI with structured output and stable exit codes
3. A maintained, reputable specialized CLI
4. An official API or SDK when no suitable CLI exists
5. An MCP server that passes the temporary headless criteria below
6. A browser or manual workflow only as a documented fallback

Prefer a CLI over MCP by default. Choose MCP only when its structured tools or
safety controls materially help and it can run temporarily:

- Without a GUI, browser, device flow, or interactive login at runtime
- From one version-pinned package, binary, or container image
- With credentials supplied through process-local environment or configuration
- Without a persistent daemon, account connection, or host-wide configuration
- Without user intervention after credential provisioning
- With clean shutdown and removable temporary files

Evaluate provenance, maintenance, task coverage, machine-readable output,
reliable exit status, least-privilege authentication, auditability, installation
cost, and separation of read-only from mutating commands.

Apply the provisioning-link safety rules to interface documentation. Prefer
first-party tools. Recommend a third-party CLI or MCP only after verifying its
source, docs, releases, and maintenance; label it third-party. Never recommend
`curl | sh`, install or start a tool while designing access, or hide mutations.

Keep secrets in `AGENT_*` variables and map them to native environment variables
for one subprocess. Do not add duplicate native secret assignments.

### 5. Write `.agent.example.vars`

Use:

```text
AGENT_<ACCESS>_<SERVICE>
AGENT_<ACCESS>_<ENVIRONMENT>_<SERVICE>
```

- Use `READ` or `WRITE` for access.
- Use `DEV` or `PROD` only at a genuine security boundary.
- Omit the environment when one credential can safely cover both.
- Never use `SHARED`; an omitted environment already means shared.

The file must:

- Contain only short comments and empty assignments.
- Group entries as `Core`, `Optional`, and `Break-glass`.
- Use active assignments for Core.
- Comment out Optional and Break-glass assignments.
- Give each credential at most one short purpose, minimum-scope, and interface
  comment.
- Exclude provisioning links, creation steps, navigation, value formats, and
  commands; put them in the final response.
- Stay near 40 lines when practical.
- Contain no real secrets, personal data, or sensitive tenant identifiers.

Example:

```dotenv
# Core

# Repository, CI, and deployment visibility via GitHub CLI; repository read-only.
AGENT_READ_GITHUB=

# Logs, metrics, traces, and alerts via Grafana API; Viewer role.
AGENT_READ_GRAFANA=

# Production database investigation via database-native CLI; read-only observer.
AGENT_READ_PROD_DATABASE=

# Reviewable branches and PRs via GitHub CLI; no administrative permissions.
AGENT_WRITE_GITHUB=

# Optional

# Direct diagnostics via provider CLI when observability is insufficient.
# AGENT_READ_PROD_PROVIDER=

# Break-glass

# Approved production repair or restore only; issue temporarily and revoke.
# AGENT_WRITE_PROD_DATABASE=
```

## Access guardrails

Prefer persistent READ access. Database READ access must use a dedicated
observer with no ownership, role administration, DDL, DML, or RLS bypass. Limit
it to required schemas or masked views and enforce read-only transactions,
timeouts, and low connection limits.

Persistent WRITE access is exceptional. Limit it to reviewable branches and
pull requests, approval-gated workflows, fixed incident notifications, or
low-budget synthetic checks. Put deployments, rollbacks, infrastructure
mutation, production database writes, and user/session mutation in Break-glass.

Share one environment-neutral credential when scoping and auditability remain
safe. Split DEV and PROD when they use different projects, production is more
sensitive, permissions cannot isolate resources, or independent revocation is
important.

Never copy runtime API keys, database owner credentials, service-role keys,
deployment tokens, signing keys, or telemetry ingestion credentials into agent
variables. Use query-capable Viewer credentials for observability.

## Final response

After writing and validating the file, respond with a concise operator
checklist. Target 350 words or fewer unless a blocker requires explanation.

1. Confirm that `.agent.example.vars` was created or updated.
2. Add `Set up now` with one numbered line per Core credential. State the
   variable, minimum scope, and verified official provisioning link when found.
3. Add `Use headlessly` with one numbered line per distinct interface. Include
   its provenance, verified setup link, minimal official install command when
   available, `AGENT_*`-to-native environment mapping, and one safe structured
   diagnostic command. For MCP, include a pinned temporary launch command and
   any important limitation. If none is safe, say so and give a manual fallback.
4. Add `Optional later` only when useful, with no more than three short items.
5. End with one reminder to store values in ignored `.agent.vars`, never in the
   example file.

Do not report services discovered, repositories inspected, selection rationale,
or successful verification steps. Mention only actions the operator must take,
important interface limitations, and failures that need attention.

Example response:

```markdown
Created `.agent.example.vars`.

Set up now:

1. `AGENT_READ_GITHUB` — Generate a repository-scoped fine-grained PAT with
   read access to metadata, contents, Actions, and deployments:
   https://github.com/settings/personal-access-tokens/new
2. `AGENT_READ_GRAFANA` — Create a Viewer service account using the verified
   official provisioning link.
3. `AGENT_READ_PROD_DATABASE` — Create a dedicated read-only observer for the
   production database.

Use headlessly:

1. GitHub CLI (official) — Install from https://cli.github.com/. Map the token
   for one process: `GH_TOKEN="$AGENT_READ_GITHUB" gh repo view --json nameWithOwner`.

Optional later: enable the commented provider credential only if telemetry is
insufficient for direct diagnostics.

Store the values in ignored `.agent.vars`; never add them to the example file.
```

## Local safety and verification

Ensure `.agent.vars` is ignored and `.agent.example.vars` remains trackable.
Never write real credential values.

After writing the template:

1. Run `bash -n .agent.example.vars`.
2. Verify active assignment names match
   `^AGENT_(READ|WRITE)_(?:(DEV|PROD)_)?[A-Z0-9][A-Z0-9_]*$` and are unique.
3. Verify no name contains `_SHARED_`.
4. Run `git diff --check`.
5. Review the diff for secrets and unrelated changes.
6. Verify every Core and Optional service has a headless interface or a concise
   explanation of why none is safe.
7. Verify READ diagnostics cannot mutate state and WRITE commands are explicit.
8. Verify interface links and provenance; pin every MCP package or image and
   confirm it satisfies all temporary headless criteria.
