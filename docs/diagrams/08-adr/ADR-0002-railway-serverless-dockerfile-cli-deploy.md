# ADR-0002 — Railway serverless in asia-southeast1, Dockerfile build, CLI deploy

- Status: accepted (retro-documented)
- Date of evidence: 2026-10-29 (owner approved serverless mode for all new services)

## Context

Service ยังไม่มี consumer จริง ค่าใช้จ่ายต้องต่ำ และ Neon อยู่สิงคโปร์ (region default us-east4 ทำให้แต่ละ query ช้าประมาณ 0.25 s ตาม ops record).

## Decision

- Railway service `logistics-api` (`https://logistics-api-production-d3bb.up.railway.app`), region `asia-southeast1-eqsg3a`, sleep on.
- Build ด้วย `Dockerfile` multi-stage (`.railway/railway.ts` `builder = "DOCKERFILE"`), healthcheck `/readyz` timeout 30 s, restart `ON_FAILURE` สูงสุด 3 ครั้ง.
- Deploy ด้วย `railway up --ci` จาก `git archive main` (CLI upload ไม่ต้องผ่าน approval gate); CI ไม่มี deploy job.

## Evidence

- `.railway/railway.ts`, `Dockerfile`
- `PRODUCT.md` (Operating Context: "Railway (`asia-southeast1`, sleeps when idle)", "Deploys are Railway CLI uploads")
- `.github/workflows/shared-runtime.yml` (image build "Nothing is pushed or deployed")
- ops record: book-platforms-status (2026-11-03) (deploy recipe, region, logistics-api-production-d3bb)

## Consequences

- Cold start: Neon resume ~6–9 s (core pool wait 20 s, connect timeout 15 s ตาม ops record).
- `SandboxCarrier` เก็บ label ledger ใน memory; การ sleep/restart ล้าง ledger (ดู 13-threat-model, 14-backup-dr).
- git/gh มองไม่เห็นการ deploy (ดู 22-dora).
- `docs/runtime.md` ยังเขียนว่า "Nothing has been deployed" — ขัดกับ `PRODUCT.md` และ ops record (เอกสาร runtime ล้าสมัย).
