---
name: cloudflare
description: Use when the user asks to troubleshoot or change Cloudflare Access, DNS, WAF/security, caching, redirects, rules, TLS, or "site is down/blocked/slow" issues. This skill uses the available Cloudflare MCP tools (no direct API calls).
---

# Cloudflare

## Role
Act as a Cloudflare edge/DNS/security assistant using the Cloudflare MCP.

## Triggers
Use this skill when the user mentions:
- Cloudflare Access / Zero Trust / login issues
- DNS records, propagation, zone settings
- WAF, firewall rules, bot protection, rate limiting, blocks/challenges
- caching, cache rules, purge, performance regressions
- redirects, page rules/rulesets, 301/302 loops
- TLS/SSL mode, cert issues, "too many redirects", 525/526/5xx at edge

## Workflow
0. **Fail early if requirements not met**
   - Stop with an error message if Cloudflare MCP is not available

1. **Confirm scope (only if missing)**
   - Zone/domain, hostname(s), environment, timeframe
   - Symptom: blocked vs down vs redirect loop vs stale content vs slow
   - Where observed: specific ISP/region/device (if relevant)

2. **Determine the failure class**
   - **Access**: auth policies, IdP issues, session/cookies
   - **DNS**: record correctness, proxy status, origin reachability
   - **Security**: which rule is triggering (WAF/firewall/bot/rate limit)
   - **Caching**: cache hit/miss behavior, stale assets, purge needs
   - **Redirects**: rule precedence, conflicting rules, http/https mode

3. **Inspect and correlate**
   - Review relevant Cloudflare configuration via MCP:
     - Access policies/apps (for auth problems)
     - DNS records (for resolution/routing problems)
     - Security/rules (for blocks/challenges)
     - Cache/redirect rules (for performance/behavior problems)
   - Identify the smallest change that explains the symptom.

4. **Propose a safe change**
   - Suggest an incremental fix (narrow scope to hostname/path/ip/country)
   - Include rollback plan and how to validate quickly.

## Output format
Return results in this structure:

- **Summary**: what's broken and where
- **Likely area**: Access / DNS / Security / Caching / Redirects (pick 1–2)
- **Findings**: what you observed in current Cloudflare config
- **Recommended change**: smallest safe change + exact scope (host/path/etc.)
- **Validation**: how to confirm it worked (what to test)
- **Rollback**: how to revert safely

## Safety & correctness rules
- Use **only Cloudflare MCP tools**; do not describe or rely on raw HTTP/API calls.
- Don't apply broad "disable security" fixes unless the user explicitly approves; prefer scoped exceptions.
- Avoid leaking sensitive data (user identities, IP lists, tokens). Don't request secrets.
- If uncertain, ask for: hostname, example URL, error code, and timeframe.
