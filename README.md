# CoLink Commerce

This repository holds two independent projects:

| Project | Folder | What it is | Status |
|---|---|---|---|
| **CoLink Commerce storefront** | repository root (`app/`, `apps/`, `prisma/`, …) | Collaborative commerce platform for e-commerce sellers and social creators, built with v0 and deployed on Vercel | Synced from v0.app |
| **Buyer Co-Pilot (Malaysia)** | [`buyer-copilot/`](buyer-copilot/README.md) | Guides people buying land or a house in Malaysia directly from the owner, with no agent. A person must approve every critical step. | Working prototype. Needs production sign-in before real use. |

The two projects share no code, dependencies or build configuration. Changes to one can't break the other.

---

## Buyer Co-Pilot at a glance

Buying property from an owner without an agent saves commission. It also removes the person who usually spots fraud, title problems and hidden costs. The Buyer Co-Pilot fills that gap without taking any decision away from the buyer:

- **Explains:** translates land office, stamp duty and conveyancing terms into plain English or Bahasa Melayu.
- **Calculates:** exact stamp duty, legal fees and cash needed, using Malaysian rates. Each figure is marked "Set by law", "Estimate" or "Proposal".
- **Flags risks:** cash hand-overs, unverified titles, caveats, Malay Reserve land, short leases, missing road access and under-declared prices.
- **Waits for you:** the purchase pauses at 5 steps, and nothing moves forward until the buyer approves. Every decision is sealed in a tamper-evident record.
- **Never acts alone:** it never moves money, signs documents or files anything with an authority.

> **General guidance, not legal advice.** Confirm every step with a licensed conveyancing lawyer.

**Stack:**
- Python engine: FastAPI with exact `Decimal` maths.
- PostgreSQL, with database-enforced approval rules and row-level security.
- n8n workflow that pauses for each human approval.
- Next.js 16 / React 19 single-page hub.
- GitHub Actions CI.

**Run it locally, without Docker** (engine and web together):

```bash
# 1. Engine (in-memory store, no database needed)
cd buyer-copilot/engine
python3 -m venv .venv && . .venv/bin/activate && pip install -e ".[dev]"
pytest                                   # 81 pass; 4 database tests skip without Postgres
export ENGINE_SERVICE_TOKEN=$(python3 -c "import secrets;print(secrets.token_urlsafe(48))")
export ENGINE_BUYER_PROXY_TOKEN=$(python3 -c "import secrets;print(secrets.token_urlsafe(48))")
export N8N_RESUME_TOKEN=$(python3 -c "import secrets;print(secrets.token_urlsafe(48))")
COPILOT_ENV=development uvicorn --factory copilot_engine.api:app_factory --port 8000 &

# 2. Web hub (same terminal, so it inherits ENGINE_BUYER_PROXY_TOKEN)
cd ../web && npm ci && npm run build
COPILOT_ENV=development COPILOT_DEMO_MODE=true ENGINE_URL=http://127.0.0.1:8000 npm start
# open http://localhost:3100
```

Full setup, including Docker Compose and n8n, plus architecture, security model, rules and CI: **[buyer-copilot/README.md](buyer-copilot/README.md)**.

---

## CoLink Commerce storefront

*Automatically synced with your [v0.app](https://v0.app) deployments*

[![Deployed on Vercel](https://img.shields.io/badge/Deployed%20on-Vercel-black?style=for-the-badge&logo=vercel)](https://vercel.com/rajs-projects-8a83d79f/v0-colinkcommerce)
[![Built with v0](https://img.shields.io/badge/Built%20with-v0.app-black?style=for-the-badge)](https://v0.app/chat/projects/otMXXHRBDpW)

This part of the repository stays in sync with your deployed chats on [v0.app](https://v0.app).
Any changes you make to your deployed app are automatically pushed to this repository from v0.app.
The system architecture is in [docs/architecture-diagram.md](docs/architecture-diagram.md).

**Deployment:** live at **[vercel.com/rajs-projects-8a83d79f/v0-colinkcommerce](https://vercel.com/rajs-projects-8a83d79f/v0-colinkcommerce)**

**Keep building:** **[v0.app/chat/projects/otMXXHRBDpW](https://v0.app/chat/projects/otMXXHRBDpW)**

**How it works**

1. Create and modify your project using [v0.app](https://v0.app)
2. Deploy your chats from the v0 interface
3. Changes are automatically pushed to this repository
4. Vercel deploys the latest version from this repository

---

## Repository layout

```
.
├── app/, apps/, components/, lib/, prisma/, …   # CoLink Commerce storefront (v0)
├── docs/architecture-diagram.md                  # storefront architecture
├── buyer-copilot/                                # Buyer Co-Pilot (self-contained)
│   ├── engine/      # Python rule engine, guardrails, approval state machine, API, tests
│   ├── db/          # PostgreSQL schema + directory seed data
│   ├── n8n/         # workflow that pauses for human approval
│   ├── web/         # one-page Buyer Command Hub (Next.js)
│   └── deploy/      # Dockerfiles, DB role bootstrap
└── .github/
    ├── workflows/buyer-copilot.yml   # CI: lint, security scans, tests, audits, SBOM, image builds
    └── dependabot.yml                # weekly dependency updates
```
