# Atryum charters for Ansible agents

One charter per agent, one agent per area of work, written in the `CHARTER.md` format from Atryum's own `populate-charter` skill (`.agents/skills/populate-charter/` in the Atryum repo), which is also the format of the judge's evaluation fixtures. Each has: header (owner, review cadence), Purpose with a success condition, Scope (in, out, handoff, **Human approval required when**, **Defer to next rule when**), a permission table with **Auto / Notify / Approve / Never** tiers and verdicts, data access and sensitive-data rules, hard stops, prompt-injection and failure defaults, the shared disruption-gate/evidence rules and over-ambition tells, a scenario table specific to the area, and a change log.

The model is a trusted junior engineer on the platform team:

| Tier | Verdict | What lands here |
|---|---|---|
| Auto | auto-approve | reads, dry-runs, idempotent converge on `ha: true` hosts in non-production, exactly the change the task spelled out |
| Notify | execute, then notify the owning channel | low-risk writes with a blast radius worth a heads-up: restarts on HA hosts, staging deploys, snapshot cleanup |
| Approve | human approval | production, `ha: false`, identity and access, network reachability, reboots, anything hard to reverse, any clause tension |
| Never | deny | destroys data, blinds monitoring or audit, widens the agent's own access, leaves the inventory |

Every charter carries the same disruption-gate and evidence rules, and the tells of an over-ambitious or compromised agent (arguments that drift from the goal, invented identifiers, self-granted access, reduced visibility, retries of denied calls, wrong environment, disproportionate volume).

| File | Agent | Collections |
|---|---|---|
| `charters/01-linux-server-config.md` | Linux server configuration | ansible.posix, community.general |
| `charters/02-cloud-provisioning.md` | Cloud provisioning | amazon.aws, azure.azcollection, google.cloud |
| `charters/03-containers-kubernetes.md` | Containers and Kubernetes | community.docker, containers.podman, kubernetes.core |
| `charters/04-network-devices.md` | Network device automation | cisco.ios, cisco.nxos, arista.eos, junipernetworks.junos, ansible.netcommon |
| `charters/05-windows-fleet.md` | Windows fleet | ansible.windows, community.windows, chocolatey.chocolatey |
| `charters/06-vmware-virtualization.md` | VMware / on-prem virtualization | community.vmware, ovirt.ovirt, community.libvirt |
| `charters/07-databases.md` | Databases | community.mysql, community.postgresql, community.mongodb |
| `charters/08-tls-pki-secrets.md` | TLS, PKI and secrets | community.crypto, community.hashi_vault |
| `charters/09-cicd-deployment.md` | CI/CD and app deployment | ansible.builtin, community.general |
| `charters/10-compliance-hardening-patching.md` | Compliance, hardening, patching | devsec.hardening, geerlingguy.*, OpenSCAP |

## Conventions the charters rely on

- **`ha: true` / `ha: false`** inventory host variable: the disruption gate. HA members take non-disruptive changes automatically; standalone hosts need a person. Same convention as the demo.
- **Environment tags**: `Environment: dev|test|staging|prod` (cloud), namespace labels (Kubernetes), folder/tag (VMware). Production is always a human decision.
- **Evidence first**: a write is only approvable when an earlier read in the same task showed the thing being changed (host exists, template exists, version is X). The judge treats identifiers that never appeared in a read as invented.
- **Windows**: a maintenance window must be named in the task for anything that patches or restarts `ha: false` hosts.
- **Secrets never appear in calls**: values come from Vault or inventory lookups; a literal password, key, or token in arguments is denied.

## Pairing with Atryum rules

Charters do the judgement; static rules do the cheap part. For each agent's MCP server, the pattern used in the demo applies: read tools -> `auto_approve`; the handful of "act" tools (launch, apply, restart) -> `ai_evaluation` with the matching charter; everything else that writes -> `auto_deny`; catch-all -> `human_approval`. Rule patterns are prefix globs, so list tool names explicitly.

## Pushing a charter

Atryum stores the charter as one text field and renders it as HTML. Push the Markdown directly (as the demo does) or convert with `../atryum/push-charter.sh`, pointing `ATRYUM_AGENT_CUID` at the agent record.
