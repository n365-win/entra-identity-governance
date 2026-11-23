# Runbooks

Azure Automation runbooks: scheduled, running on a managed identity, delivering findings by email.

## Why these exist alongside `scripts/`

`scripts/` holds the interactive tools; run them by hand when investigating.

These run the same logic on a schedule. The code is the same and the delivery is what differs. A CSV on somebody's desktop requires that person to be at that machine, to remember, and to look. A runbook runs whether anyone remembers or not. The CSV is the evidence; the email is what makes it a control.

## Schedules

| Runbook | Cadence | Why |
|---|---|---|
| `Runbook-DuplicateCheck` | 15 to 20 min after HR-Sync | Provisioning is asynchronous; running immediately after upload sees nothing |
| `Runbook-AccessChangeWatch` | Daily, or hourly for high-sensitivity groups | Direct group additions are silent otherwise |

## Automation variables

Both reuse what HR-Sync already has: `AlertTo`, `GmailAddress`, `GmailAppPassword`.

## Managed identity permissions

| Runbook | Graph permissions |
|---|---|
| `Runbook-DuplicateCheck` | `User.Read.All` |
| `Runbook-AccessChangeWatch` | `AuditLog.Read.All`, `Group.Read.All`, `User.Read.All`, `Mail.Send` |

`Mail.Send` is an application permission, so it lets the identity send as any mailbox in the tenant. Scope it with an application access policy to the single sending mailbox, or accept that it is broader than it looks.

## Alerting behaviour

Both call `Write-Error` when they find something, which marks the Automation job as failed. An Azure Monitor alert rule on job failure then catches the finding even if SMTP is blocked from the sandbox. That gives two independent delivery paths, so a silent SMTP failure does not swallow the finding.

`Runbook-DuplicateCheck` alerts only on tiers 0, 1 and 2. Tier 4 is two real people sharing a name, and emailing about it every run teaches people to ignore the alert.
