# 18 — Git and release workflow

จาก `git log` (90 วัน): งานเข้าทาง branch → PR หรือ merge commit ลง `main` (PR #2–#13 merged 2026-11-02/03; ก่อนหน้านั้น merge commit จาก branch เช่น `logistics-runtime`, `build-hygiene`, `invalid-input`, `sentry`). ไม่มี tag หรือ GitHub release. Deploy แยกจาก git: owner/agent รัน `railway up --ci` จาก `git archive main`.

```mermaid
gitGraph
  commit id: "e4a5f09 scaffold"
  commit id: "a25adf3 schema rename"
  branch logistics-runtime
  commit id: "ee04ddc runtime pilot"
  commit id: "8fed213 review fixes"
  checkout main
  merge logistics-runtime id: "033af3f"
  branch build-hygiene
  commit id: "7143b0b image + SHA pins"
  checkout main
  merge build-hygiene id: "961c821"
  branch sentry
  commit id: "507cce5 Sentry"
  checkout main
  merge sentry id: "45edc2c"
  branch fix-sentry-hardening
  commit id: "1f05d6c port hardening"
  checkout main
  merge fix-sentry-hardening id: "1f86052 PR 2"
  branch no-free-text
  commit id: "3942a64 no free text"
  checkout main
  merge no-free-text id: "501601f PR 12"
  commit id: "96f23af uv + mypy + coverage"
```

```mermaid
flowchart LR
  B["Feature branch"] --> P["PR or local merge"]
  P --> CI["check.yml + shared-runtime.yml on push and PR"]
  CI --> M["main"]
  M --> A["git archive main"]
  A --> U["railway up --ci (no approval gate for CLI uploads)"]
  U --> H["Railway healthcheck /readyz"]
  H --> V["Verify /readyz 200 + sentry enabled log"]
```

Sources: `git log --since=90.days origin/main`, `gh pr list -R bookchaowalit-backend/platform-logistics --state all`, `.github/workflows/check.yml`, `.github/workflows/shared-runtime.yml`, `railway.toml`, ops record: book-platforms-status (2026-11-03) (deploy recipe, "CLI uploads need no approval", "/readyz 200, `sentry: enabled` logged")

ชื่อ branch ใน gitGraph ย่อมาจาก merge message (เช่น "Merge Sentry error reporting (sentry)") หรือ `gh pr list`: `fix-sentry-hardening` = `fix/sentry-hardening-port` (PR #2), `no-free-text` = `fix/sentry-no-free-text` (PR #12). PR #3–#11 (`fix/sentry-*`) และ #13 (`ci/uv-lock-mypy-product`) ไม่ได้วาดทีละเส้น.

## Open questions

- PR #1 (open, "port the hardened scrubber") ถูกแทนที่แล้ว — ปิดได้หรือไม่ (ops record: "superseded PRs from branch `sentry-hardened-scrubber`")
- จะมี tag/release ต่อ deploy หรือไม่ เพื่อให้วัด DORA ได้
