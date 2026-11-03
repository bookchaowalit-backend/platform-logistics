# ADR-0004 — platform-identity RS256 JWT: workspace from `ws`, per-route scopes

- Status: accepted (retro-documented)
- Date of evidence: 2026-10-29 (owner decision: services trust the platform-identity issuer); activated 2026-11-03

## Decision

- `TokenVerifier(issuer=JWT_ISSUER, audience=JWT_AUDIENCE, public_key=JWT_PUBLIC_KEY | jwks_url=JWT_JWKS_URL)`; ต้องมีอย่างใดอย่างหนึ่ง.
- Production: issuer `https://identity-api-production-fad1.up.railway.app`, JWKS `/.well-known/jwks.json`, audience `logistics-api`.
- Tenant มาจาก claim `ws` เท่านั้น; ทุก route ใช้ `require_scope(...)`: `rates:read`, `shipments:read`, `shipments:write`, `labels:write`, `tracking:write`.

## Evidence

- `src/platform_logistics/__main__.py`, `src/platform_logistics/app.py`
- `runtime_tests/conftest.py` (claims), `runtime_tests/test_http.py` (`test_rates_need_scope_and_are_deterministic`)
- `PRODUCT.md` ("Tokens come from platform-identity (audience `logistics-api`)")
- `docs/runtime.md` ("Workspace authority comes only from the JWT `ws` claim")
- ops record: book-platforms-status (2026-11-03) (JWT line, Identity activated)

## Consequences

- token จาก `empire-platform` ได้ 401 (ops record).
- person token ยังใช้ไม่ได้จนกว่า owner ตั้ง `IDENTITY_GOOGLE_CLIENT_IDS`.
