# Privacy & Security

This framework demonstrates security patterns at the architectural level while
running on free-tier infrastructure. That combination creates real gaps, and
they are documented here rather than implied away.

The distinction that matters throughout: **controls that are implemented**,
**controls that are designed but not implementable on the current tier**, and
**controls that are genuinely absent**. An enterprise deployment must close the
second and third categories before handling production data.

---

## Summary of Controls

| Domain | Control | Status |
|---|---|---|
| Least privilege | Scoped IAM policies per identity, single bucket and prefix | Implemented |
| Least privilege | Read-only access for the analytics platform | Implemented |
| Least privilege | Medallion layers as separate catalogs, granted independently | Implemented |
| Least privilege | Grants defined as code | Gap — no account API on free tier |
| Credentials | No secrets in version control; scanning and pre-commit hooks | Implemented |
| Credentials | No secrets in application code; connections referenced by ID | Implemented |
| Credentials | No secrets in Terraform state; keys created out of band | Implemented |
| Credentials | Assumed role with external ID for platform-to-storage access | Implemented |
| Credentials | Finite token lifetimes | Implemented |
| Credentials | Secrets manager with automated rotation | Gap — not yet adopted |
| Network | No public ingress; all local services bound to the host | Implemented |
| Network | S3 public access blocked, encrypted at rest, versioned | Implemented |
| Network | IP allow-listing on the data platform | Gap — no network config on free tier |
| Network | Private connectivity for warehouse traffic | Gap — enterprise tier only |
| Identity | Token-based access with expiry, no passwords | Implemented |
| Identity | Federated SSO through an identity provider | Gap — no IdP integration (expensive)|
| Identity | Multi-factor authentication | Gap — single factor only |
| Identity | Service principals for automation | Gap — unavailable on free tier |
| Data protection | Classification applied as catalog metadata, through reviewed code | Implemented |
| Data protection | Row-level lineage to source file and load time | Implemented |
| Data protection | Encryption at rest and in transit | Implemented |
| Data protection | Column masking, row filtering, tokenization | Gap — no sensitive data present |

## Summary of Gaps

Ordered by what an enterprise deployment must address first.

| # | Gap | Cause | Resolution |
|---|---|---|---|
| 1 | No IdP or MFA | Local, single-operator | Federate through Okta or Entra ID |
| 2 | Databricks open to any network | Free tier has no network config | IP access lists, PrivateLink |
| 3 | Long-lived AWS keys | Docker has no cloud identity | Deploy to a platform with instance roles |
| 4 | Human identity used for automation | No service principals on free tier | Service principal with OAuth |
| 5 | No masking, filtering, or tokenization | No sensitive data present | Attach policies to existing tags |
| 6 | Grants applied by hand | No account API on free tier | Databricks Terraform provider |
| 7 | No secrets manager | Not yet adopted | Airflow secrets backend |
| 8 | No credential rotation | Manual process | Scheduled rotation, short-lived tokens |

**Nothing in this framework should handle production or personal data without
resolving items 1 through 5.** The architectural patterns are sound and the
governance metadata is in place. The enforcement layer is what is missing, and
it is missing because the free tier does not expose it — not because it was
overlooked.

The sections below detail each domain.

---

## Least-Privilege Access

Access is granted by function, not by convenience. Every identity in the
framework holds the narrowest set of permissions that lets it do its job.

### Implemented

| Identity | Grants | Scope |
|---|---|---|
| Airflow → AWS | `GetObject`, `PutObject`, `ListBucket` | Landing prefixes of one bucket |
| Databricks → AWS | `GetObject`, `GetObjectVersion`, `ListBucket` | Read-only, landing prefixes only |
| Databricks compute | `USE CATALOG`, `USE SCHEMA`, `SELECT`, `MODIFY` | Named catalogs and schemas |
| External locations | `READ FILES` | Scoped to a single S3 prefix |

Two design choices reinforce this:

- **`s3:ListBucket` is constrained by prefix condition**, not granted at bucket
  scope. Listing is a bucket-level action in IAM, so a `StringLike` condition
  on `s3:prefix` is the only way to narrow it.
- **Medallion layers are separate catalogs** (`raw`, `transform`, `formal`).
  Unity Catalog grants are per-catalog, which makes "consumers read `formal`
  and nothing else" a single grant rather than a policy exercise.

### Gap

**Grants are applied by hand, not defined as code.** Unity Catalog objects are
created by SQL run once, so there is no reviewable record of who was granted
what and no drift detection. On a paid tier the Databricks Terraform provider
would manage storage credentials, external locations, and grants alongside the
AWS resources.

**No role separation between human and automated identities.** A single
personal identity performs both interactive analysis and pipeline execution,
so audit logs cannot distinguish them.

---

## Credential Management

### Implemented

- **No secrets in version control.** `.gitignore` covers `.env`, `*.tfvars`,
  `profiles.yml`, and `.databrickscfg`. `detect-secrets` and
  `detect-private-key` run as pre-commit hooks, and GitHub secret scanning with
  push protection is enabled on the repositories.
- **No secrets in application code.** Connections are referenced by ID
  (`aws_default`, `databricks_default`) and resolved at runtime. Credentials
  never appear in a DAG, a model, or a log line.
- **No secrets in Terraform state.** IAM access keys are deliberately not
  managed by Terraform — an `aws_iam_access_key` resource writes the secret
  into state in plaintext. Keys are created out of band.
- **Assumed roles where the platform allows it.** Databricks reaches S3 through
  an IAM role with an external ID condition. Nothing secret is exchanged; STS
  issues short-lived credentials. The external ID prevents another tenant of
  the same SaaS platform from naming the role ARN in their own workspace.
- **Finite token lifetimes.** Databricks personal access tokens are issued with
  an expiry rather than set to never expire.

### Gap

**Long-lived static credentials exist.** Airflow runs in local Docker, which
has no cloud-native identity, so an IAM user with access keys is the only
option available. Those keys sit in a `.env` file, do not rotate, and would
work from anywhere if copied.

The mitigation is scope, not elimination: the keys reach one bucket and two
prefixes. The fix is deployment — on managed Airflow, EKS, or EC2, the user is
deleted and an instance role takes its place with no code change.

**No secrets manager.** Credentials are environment variables rather than
values resolved from AWS Secrets Manager or Databricks secret scopes at
runtime. Airflow supports a secrets backend natively; adopting one is a
configuration change, not a redesign.

**No automated rotation.** Key and token rotation is manual and unscheduled.

---

## Network & Application Access

### Implemented

- **No public exposure.** Airflow, Kafka, and the dbt tooling run entirely in
  local Docker. Nothing listens on a public interface, and there is no ingress
  path from the internet.
- **S3 is closed by default.** Public access block is enabled on all four
  settings, bucket policies grant no anonymous access, and server-side
  encryption is enforced.
- **Object versioning is enabled**, so an accidental overwrite or deletion is
  recoverable.

### Gap

**Databricks is reachable from any network.** The workspace accepts
authenticated connections from any source IP. In an enterprise deployment this
is closed with **IP access lists**, restricting the workspace to corporate
egress ranges and VPN exit points. Free Edition does not expose account-level
network configuration, so the control cannot be applied here.

The enterprise pattern would layer three things:

1. **IP access lists** on the workspace — an allowlist of known egress CIDRs,
   with an explicit denylist taking precedence.
2. **Private connectivity** (PrivateLink or equivalent) so warehouse traffic
   never traverses the public internet.
3. **VPC endpoint policies** on S3, so the bucket is reachable only from the
   expected network path regardless of what credentials are presented.

**Local deployment is not a security control.** It limits exposure by
accident — because nothing is deployed — rather than by design. A deployed
version of this framework needs the network controls above from the first day,
not retrofitted.

---

## Identity & Authentication

### Implemented

- **Local Airflow authentication is disabled deliberately**, not by oversight.
  The stack is unreachable from outside the host, and a login screen on a
  single-operator local container adds friction without adding protection.
  This is documented as dev-only.
- **GitHub access uses tokens with expiry**, not passwords.

### Gap

**No identity provider.** Authentication is per-application and locally
managed. An enterprise deployment federates through an IdP — Okta, Entra ID,
or equivalent — and that changes several properties at once:

| Property | Today | With an IdP |
|---|---|---|
| Credential | Per-app token or password | Federated SSO |
| MFA | None | Enforced centrally |
| Provisioning | Manual | SCIM, group-driven |
| Deprovisioning | Manual, per-app | Single revocation removes all access |
| Audit | Per-app logs | Central sign-in log |

Deprovisioning is the one that matters most. Without an IdP, revoking a
departing user's access means remembering every system they touched. With one,
it is a single action — and that difference is the reason IdP integration is
not optional at enterprise scale.

**No service principals.** Automation authenticates as a human identity
because Free Edition provides no account console and therefore no service
principals. Every pipeline action is attributed to a person in the audit log,
and the credential dies with that person's account. Unity Catalog grants are
already written as least-privilege, so substituting a service principal on a
paid tier is a credential change rather than a redesign.

**No MFA on the data platform.** A personal access token is a single factor.

---

## Data Classification & Protection

### Implemented

- **Classification is applied as catalog metadata.** Every `formal` dataset
  carries governance tags — `informationClassification`,
  `complianceClassification`, `legalHold`, `dataOwner`, `dataPrimacy` — applied
  through dbt in a reviewed pull request rather than by hand in a console.
  Tags are queryable through `system.information_schema.table_tags`, so
  "what is classified how" is answerable in SQL.
- **Lineage is traceable to the row.** Every record carries the source file and
  load timestamp that produced it, so any value can be traced to its origin.
- **Encryption at rest and in transit** is enabled on S3 and inherent to
  Databricks-managed storage.

### Gap

**No anonymization, masking, or tokenization is implemented.** The datasets in
this framework are entirely public — NYC TLC trip records and MTA transit
feeds contain no personal data, and every dataset is classified `PUBLIC` with
`complianceClassification` of `NONE`.

This is an honest limitation rather than a deliberate omission: there is
nothing here to protect, so the controls have nowhere to demonstrate
themselves. An enterprise deployment handling personal data would need:

- **Column masking** on sensitive fields, applied through Unity Catalog masking
  functions so the control travels with the column rather than living in
  each query
- **Row filtering** so access to a table does not imply access to every row
- **Tokenization or hashing** of direct identifiers before they reach the
  `formal` layer
- **Differential handling by classification** — the tagging taxonomy already
  exists and is the natural place to drive these policies, since Unity Catalog
  supports attribute-based access control keyed on tags
- **Retention and right-to-erasure** procedures, which the
  `dataDurationClassification` and `legalHold` tags anticipate but do not
  enforce

The tagging framework is deliberately built to carry these policies. It is the
hook; the policies are not attached to it.