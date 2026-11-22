# Self Hosted n8n: What Server Specs You Actually Need, What Breaks After the First Install, and Which Plan to Buy (Full Plan Table and Cost Math)

Self hosted n8n runs on 2 vCPU and 4 GB of RAM at the floor, on a Linux machine with Docker Compose v2, and the official compose stack takes about 15 minutes to bring up if nothing goes wrong. On Ava Hosting's price list that machine is €10.00/month. If you want the one-click route instead of hand-writing compose files, the brand's app catalogue lists n8n from €5/month with PostgreSQL provisioned alongside it.

What follows is the part most tutorials leave out: which of the four install paths still makes sense, why 4 GB is the real floor rather than a comfortable suggestion, the eight things to change after `docker compose up -d`, and what this costs over three years instead of one month.

## The short version: which plan to pick

> Most readers should start on the 2 vCPU / 4 GB tier at **€10.00/month**. It meets n8n's documented minimum without sitting exactly on it, leaves room for PostgreSQL next to the app, and stays cheap enough that a failed experiment costs you one month. Step up to 8 GB only if you plan to run n8n Assistant code sandboxes, a busy webhook endpoint, or several concurrent long-running workflows.
>
> 👉 [See the 2 vCPU / 4 GB VPS plan](https://my.ava.hosting/aff.php?aff=15562)

Two product lines are on the site and they are not the same thing. The VPS line gives you root access, a control panel, weekly backups, DDoS protection and a 1 Gbps port, which is what you need to run your own Docker Compose file. The self-hosted app line is the smaller, simpler set of plans; the €2 tier lists only 1 GB of RAM, which is below what n8n needs before you add a database to the same box.

## Every Ava Hosting plan currently on the price list

VPS plans, all quoted as "starting from" a monthly figure:

| Plan | vCPU / RAM | NVMe | Price | Order |
| --- | --- | --- | --- | --- |
| VPS Nano | 1 vCPU / 512 MB ECC | 20 GB | €11.00 / year (listed as 0 available) | [VPS Nano](https://my.ava.hosting/aff.php?aff=15562) |
| VPS One | 1 vCPU / 2 GB ECC | 25 GB | €5.00 / month | [VPS One](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Basic | 2 vCPU / 4 GB ECC | 50 GB | €10.00 / month | [VPS Basic](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Lucky | 2 vCPU / 6 GB ECC | 70 GB | €15.00 / month | [VPS Lucky](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Power | 4 vCPU / 8 GB ECC | 90 GB | €20.00 / month | [VPS Power](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Extra | 6 vCPU / 12 GB ECC | 120 GB | €30.00 / month | [VPS Extra](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Ultra | 8 vCPU / 16 GB ECC | 160 GB | €40.00 / month | [VPS Ultra](https://my.ava.hosting/aff.php?aff=15562) |

Self-hosted app plans:

| Plan | vCPU / RAM | NVMe | Price | Order |
| --- | --- | --- | --- | --- |
| Start | 1 vCPU / 1 GB | 15 GB | €2.00 / month (free trial listed) | [Start](https://my.ava.hosting/aff.php?aff=15562) |
| Mini | 1 vCPU / 2 GB | 25 GB | €5.00 / month | [Mini](https://my.ava.hosting/aff.php?aff=15562) |
| Basic | 2 vCPU / 4 GB | 50 GB | €10.00 / month | [Basic](https://my.ava.hosting/aff.php?aff=15562) |
| Pro | 4 vCPU / 8 GB | 100 GB | €20.00 / month | [Pro](https://my.ava.hosting/aff.php?aff=15562) |
| Business | 6 vCPU / 12 GB | 150 GB | €30.00 / month | [Business](https://my.ava.hosting/aff.php?aff=15562) |
| Power | 8 vCPU / 16 GB | 200 GB | €40.00 / month | [Power](https://my.ava.hosting/aff.php?aff=15562) |

Everything on the VPS line includes unlimited bandwidth, a 1 Gbps port, free DDoS protection, weekly backups, full root access, a control panel, a 99.9% SLA and 24/7 support, running on Intel Xeon E5 hardware with NVMe storage. Port 25 is closed on VPS Nano, which matters only if you plan to send mail straight from n8n.

One thing to verify at checkout rather than assume: the prices above are display prices. Whether your term is monthly or annual, and what the renewal price is after the first period, is shown on the order page for the specific configuration you select. Nothing on the public plan list states a first-year discount, so budget the listed figure as your ongoing cost.

## Four ways to install n8n, and the one the docs no longer recommend

The official deployment page lists five entry points: one-line setup, Docker Compose, a cloud provider (AWS, Azure, DigitalOcean, Hetzner, Heroku, OpenShift), and npm.

**One-line setup** is the fastest path. It needs Linux or macOS with curl installed and handles dependencies for you. Use it if you want n8n running today and don't care about controlling the container layout.

**Docker Compose** is what you want for anything you'll keep. It's a multi-container setup with persistent data, and it's the only path where you can see and edit every service n8n actually runs. Requirements are Docker plus Docker Compose v2, which you can confirm with `docker compose version`.

**Cloud platform images** exist for AWS, Azure, DigitalOcean, Hetzner, Heroku and OpenShift. These are fine if you already live in one of those accounts. You pay for the convenience in either money or configuration time.

**npm** is deprecated from n8n 3.0. The docs now say so directly and point you at Docker Compose or the one-line setup instead. Community tutorials from 2025 still call npm "the simplest way to self-host locally," and that advice has expired. Node.js version management and dependency drift are the reasons it lost.

For a persistent instance that answers webhooks at 3am, you want a Linux box you control, with Docker on it. That is the entire argument for a VPS over a spare laptop.

👉 [Pick a Docker-ready VPS with root access](https://my.ava.hosting/aff.php?aff=15562)

## How much server n8n really needs, and why 4 GB is the floor

The documented minimum is 2 vCPU and 4 GB of RAM, and heavier workloads want more cores and memory. The official Docker Compose guide repeats the same numbers and explains where they come from: the stack includes `sandbox-runner-1`, a privileged Docker-in-Docker container that creates and runs sandboxes when n8n Assistant needs to execute code. DinD needs more headroom than an ordinary container, and that headroom is baked into the 4 GB figure.

That detail changes the sizing conversation. If you follow an older guide that spins up a single n8n container, 2 GB looks adequate and will run for a while. If you use the current official compose file, you're running five services:

- `n8n`, the editor at `http://localhost:5678`
- `sandbox-certs`, which generates TLS certificates once and exits
- `sandbox-api`, the control plane for code execution
- `sandbox-runner-1`, the privileged DinD container doing the actual work
- `searxng`, the bundled web search backend

The call chain runs n8n → `sandbox-api` → `sandbox-runner-1` → a fresh sandbox container per execution. SearXNG needs `search.formats: [html, json]` in its settings file, because the default image only emits HTML and n8n's web search needs JSON.

If you're on Windows, run this inside WSL, not from Windows directly. Keep the project in the WSL filesystem, something like `~/n8n`, and never under `/mnt/c/...`. Cross-boundary bind mounts are slow and produce permission errors that look like n8n bugs.

The database decision is separate. The default install runs SQLite, and the official compose example has no database service at all, so n8n falls back to its built-in SQLite file. SQLite is fine for a single user and internal tooling. For production, multi-user instances and any meaningful execution history, switch to PostgreSQL. That adds a service to the box, which is the second reason not to buy the smallest plan.

👉 [Compare the 4 GB and 8 GB tiers side by side](https://my.ava.hosting/aff.php?aff=15562)

## The post-install checklist most tutorials skip

Getting the container up is the easy half. These are the items that decide whether the instance survives its first month.

**The volume is the whole installation.** Workflows, credentials and n8n's encryption key live in `/home/node/.n8n`. Mount it as a named volume and your data survives container recreation. Lose it and you lose the encryption key, which means every stored credential becomes unreadable. Back up the volume, not just the database.

**Port 5678 is the default.** If you change the host port mapping, that's the port you actually open in the browser. People following screenshot tutorials hit connection refused because they visit the port from the guide instead of the one they mapped.

**PostgreSQL 18 needs `PGDATA` set.** The official example uses the `postgres:18` image and sets `PGDATA=/var/lib/postgresql/data`. Skip it and Postgres 18 writes outside the mounted volume, so the database starts empty and looks broken. Also: upgrading an existing older Postgres straight to 18 fails with `database files are incompatible with server`. Back up with `pg_dumpall` first and follow the official upgrade guide.

**Existing SQLite data does not migrate itself.** Configuring the Postgres environment variables gives you a clean new instance. The n8n side runs its migrations on startup, but the workflows sitting in your SQLite file stay there.

**Three environment variables now produce deprecation warnings on first boot.** `DB_SQLITE_POOL_SIZE` should be set above zero, because running SQLite without a read connection pool is deprecated. `N8N_RUNNERS_ENABLED=true` turns on task runners before they become the default. And `N8N_BLOCK_ENV_ACCESS_IN_NODE` will flip from `false` to `true`, so if any Code node or expression reads environment variables, set it explicitly to `false` now.

**Serve it over HTTPS.** Plain HTTP on a public IP leaks credentials in transit. The community compose example uses a reverse proxy for this, and any TLS-terminating proxy in front of 5678 will do.

**Keep the encryption key out of `.env`.** On a shared or externally reachable box, a plaintext key in an environment file is a credentials leak waiting for someone to scan your source tree. A secret manager is the safer home for it.

**Treat `sandbox-runner-1` as root on the host.** It runs `privileged: true` with Docker-in-Docker. It must never be reachable from the public internet, and your cloud firewall should open only n8n's port. `SANDBOX_API_KEYS`, the registration token and the runner key each need unique values, no leftover `change-me-...` strings, and periodic rotation. The `sandbox-api` and runner services deliberately do not load `env_file: .env`, so model API keys, the Brave key, the Postgres password and n8n's encryption key never reach a sandbox container. The mTLS keys in the `sandbox-tls` volume, including the root CA key, sit at `0600`.

**Certificates from `sandbox-certs` do not renew.** You need a plan for regenerating the `sandbox-tls` volume before expiry, or the sandbox services stop talking to each other on a date you didn't diarise.

Two smaller points worth knowing before you debug something that isn't broken. n8n Assistant is off by default and only activates once you configure a model, either in the instance AI settings or in `.env`. And for production instances, the docs currently recommend Daytona as the sandbox provider rather than the bundled `n8n-sandbox`, which is aimed at local development and testing.

Common failures map to predictable causes: sandbox services failing on certificate errors means `sandbox-certs` didn't finish; a crash loop complaining that something `must be set` means `SANDBOX_RUNNER_API_KEYS` or `SANDBOX_RUNNER_REGISTRATION_TOKEN` is missing; sandbox calls failing from n8n means the URL or key in `.env` doesn't match `sandbox-api`.

## What self hosting actually costs over three years

Everything except licensing and your own time is the server. n8n itself is free: without a license key it runs as the Community edition, and Business or Enterprise features require the corresponding key. The licence is fair-code rather than open source, which means personal and internal business workflows are free, while building it into a consumer-facing product you sell requires a commercial licence.

At the €10.00/month tier, year one is €120 and every year after is €120. At €20.00/month it's €240 a year. The €5 app-hosting tier is €60 a year if a single 1 vCPU / 2 GB box running n8n plus PostgreSQL covers your workload. That's the whole cost model: infrastructure, fixed, whether you run ten workflows or ten thousand.

Compare that to per-execution billing, where the invoice moves with volume, and the crossover usually arrives faster than people expect once a workflow polls an API every few minutes. It also means no ceiling on active workflows, users or execution time, no noisy neighbours on shared compute, and the freedom to install community nodes or a private internal node.

The counterweight is your time. A single instance behind a proxy is a weekend project. A production instance with uptime expectations is ongoing work: persistent volume claims, a managed PostgreSQL to patch, ingress and TLS, monitoring, and manual version upgrades that need release notes read before you press go.

## When self hosting n8n is the wrong call

Skip it if any of these describe you.

You don't want to touch Docker or a terminal, and "SSH" is a word you'd rather not learn. The one-click app route exists for exactly this, but someone still owns upgrades and backups.

You need high availability for workflows that genuinely cannot miss a run. A single VPS is a single point of failure regardless of how much RAM you buy.

You're running high-volume workflows that need multiple workers, a Redis queue and a load balancer. That's a Kubernetes conversation, and the configuration overhead is real.

You need a named person to blame when a third-party API changes shape at 2am. Self hosting moves that responsibility onto you.

## Picking your plan by scenario

Personal automations, a handful of workflows, one user: the €5 tier is enough if you're on the app-hosting line, and VPS One at €5/month is enough if you want root and your own compose file. Neither leaves room for a comfortable PostgreSQL setup, so keep the SQLite default and back up the volume.

A team instance with PostgreSQL, webhooks that outside services depend on, and more than one person editing workflows: VPS Basic at €10.00/month, 2 vCPU and 4 GB. This is the plan the documented minimum actually points at.

n8n Assistant turned on with code execution, or file-heavy workflows, or several long-running jobs at once: VPS Power at €20.00/month for 4 vCPU and 8 GB. The sandbox runner is the reason, and 8 GB is where that stack stops competing with itself.

Before you order, confirm two things on the order page: that the plan you picked gives you root access and Docker (the VPS line does, and that's what the official compose file assumes), and what the renewal price and billing term are for the configuration you selected. Then check the first boot logs for the three deprecation warnings, set the volume mount, and you're done with the hard part.

👉 [Open the Ava Hosting plan page and pick your tier](https://my.ava.hosting/aff.php?aff=15562)
