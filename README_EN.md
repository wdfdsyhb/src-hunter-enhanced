# src-hunter-enhanced · src-hunter, Enhanced for Chinese SRCs

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE) ![ZCode Skill](https://img.shields.io/badge/AI-ZCode_Skill-purple)

> Give it a target and it runs the five-stage hunt pipeline: intake → recon → enum → hunt → report. 19 playbooks, 305 structured payloads, 2887 disclosed HackerOne cases — this fork adds SaaS tenant-bypass, edu/gov playbooks and recent high-frequency CVE payloads.

> **This repository is an enhanced fork** of [MyuriKanao/src-hunter](https://github.com/MyuriKanao/src-hunter) (MIT License).
> Additions over upstream (2026-10):
>
> | Change | Description |
> |------|------|
> | `references/experience/cn-saas-tenant-bypass.md` | SaaS multi-tenant broken-authorization playbook: the four-breakpoint model, zero-credential guest access, three-way tenant-parameter substitution, two-way closed-loop stop-loss |
> | `references/industry/edu-gov.md` | Education / gov-enterprise playbook: the Butian public-testing main battleground, asset-scope inference, education-SaaS vulnerability mapping |
> | `references/dictionaries/chinese-srcfingerprints.md` new section 1.7 | Education SaaS fingerprints (Chaoxing family / online exams / university one-stop services / academic affairs) |
> | `references/payloader/by-category/web/近年高频CVE-2023至2024.md` | Fills the post-2022 payload gap (PHP-CGI / OFBiz / Confluence / GitLab / WebLogic / Hikvision, etc.) |
> | SKILL.md routing table | Phases 3/4 wired to the new files above |
>
> The upstream knowledge base (19 playbooks / 305 payloads / H1 cases / WooYun statistics) is unchanged. Compliance boundaries and anti-hallucination discipline match upstream.

# src-hunter

A Claude Code skill for SRC programs, crowdsourced testing and bug bounty.

In short: you give it a target, and it advances vulnerability hunting through a fixed pipeline — confirm the target's scope first, then recon and asset enumeration, then vulnerability testing, then report writing.

```text
intake → recon → enum → hunt → report
```

The project ships curated knowledge bases assembled from public sources, including:

- 19 attack playbooks
- 305 structured payloads
- WAF / EDR bypass variants
- HackerOne disclosed High / Critical hacktivity data
- Statistical residue from the WooYun historical archive
- Fingerprints and default credentials of common Chinese components

## Installation

Marketplace:

```bash
/plugin marketplace add MyuriKanao/src-hunter-skill
/plugin install src-hunter@src-hunter
```

Git:

```bash
git clone https://github.com/MyuriKanao/src-hunter-skill.git ~/.claude/skills/src-hunter
```

## Directory Layout

```text
references/
  methodology/    five-stage pipeline, attack prioritization, bypass toolset, evidence rules
  playbooks/      one file per vulnerability class, with real H1 cases and payloads
  industry/       vertical playbooks for banking/finance, telecom/ISP
  dictionaries/   fingerprints and default credentials of Chinese components
  templates/      CVSS 4.0 report templates
  h1-reports/     raw data of 2,887 disclosed reports, grouped by weakness
  payloader/      305 structured payloads, 263 WAF/EDR bypass steps, 114 tool commands
```

Playbooks are the main entry point. All playbooks are written from a black-box perspective: by default you have a URL, not the source code.

Every playbook revolves around the same set of questions:

- where to find entry points
- which payloads to test with
- which response characteristics to watch
- how to judge impact
- how to raise the report's value
- what must not be done

The idea is not to pile up payloads, but to chain testing actions, evidence retention and report output.

## MCP Tool Integration

The skill integrates a local MCP server as its tool layer, so during the hunt phase Claude can directly call browser automation, CDP debugging, network interception, JS hooks, AST deobfuscation, Frida memory verification, WASM reversing, source map reconstruction, Android adb bridging, SSL pinning bypass and more.

**Current main choice**: [jshookmcp](https://github.com/vmoranv/jshookmcp) 0.3.0 (134 curated tools / 386 total / 36 domains); the full index and scenario mapping are in [`references/tools/mcp-jshook.md`](references/tools/mcp-jshook.md).

7 highly related playbooks (`xss` / `rce` / `ssrf-cache-host` / `mobile` / `oauth-saml-jwt` / `api-rest` / `file-upload`) each end with a `## 相关 MCP 工具` (Related MCP tools) back-anchor, naming which jshook tools to call for that attack surface and when.

## TODO

- Support more tools
- Multi-agent execution workflow

## Trigger Keywords

Built-in trigger words include:

- bug bounty, HackerOne, SRC hunting, vulnerability bounty, crowdsourced testing
- WAF bypass
- how to test an endpoint / API / parameter
- arbitrary account, arbitrary modification, arbitrary deletion
- password reset, account recovery
- default credentials, Actuator, exposed admin panels

You can also invoke it explicitly:

```text
/src-hunter:src-hunter <target>   # Marketplace plugin install
/src-hunter <target>              # Plain git standalone skill
```

## Playbook List

| Playbook | Embedded H1 cases |
|---|---:|
| arbitrary-x-authz (IDOR / arbitrary account / privilege escalation) | 465 |
| rce (deserialization / SSTI / XXE / frameworks) | 385 |
| xss | 335 |
| info-disclosure | 319 |
| oauth-saml-jwt | 240 |
| logic-flaws (CSRF / clickjacking / payments) | 234 |
| path-traversal / LFI / RFI | 163 |
| sqli | 147 |
| dos | 138 |
| ssrf-cache-host | 108 |
| unauth-access (default credentials / Actuator / exposed services) | 46 |
| http-smuggling / CRLF | 38 |
| api-rest / WebSocket | 15 |
| file-upload | 8 |
| mobile (Android / iOS) | 8 |
| race-conditions | 5 |
| llm-prompt-injection | 1 |
| graphql | 1 |
| intranet-postexp (intranet / post-exploitation quick reference) | — |

## Data Sources

- HackerOne hacktivity feed: 2,887 disclosed High / Critical reports, from public data.
- WooYun historical archive: covers 88,636 cases; only statistical residue is kept — parameter frequencies, case IDs and bypass patterns.
- Payloader: 305 structured payloads + 263 WAF / EDR bypass steps + 114 tool commands; the original repository is `3516634930/Payloader`.

This project only organizes, translates and reassembles public material. It contains no proprietary data and scrapes nothing behind authentication.

## Red Lines

Every playbook ends with concrete boundaries; these are the ones most commonly tripped:

- **Sample control**: proving SQLi with the database name / version is enough — don't dump data; for IDOR and Mongo / ES, 1–3 sample records suffice, never full pulls.
- **Test with your own accounts**: authorization bypass, password reset, JWT forgery, redirect_uri and blind XSS are all tested between two accounts you registered yourself. **Never touch strangers' accounts** — even if you can.
- **Read-only, no writes**: with RCE run only `id` / `whoami` / `uname -a`; on unauthenticated Redis / Mongo only `info` / `ping` / `db.version()`; on arbitrary file read, stop at the first `root:x:` line — don't read `/etc/shadow`.
- **No real side effects**: no real SMS, no real charges, no real email, no real refunds, no real file overwrites, no real notice / email-template changes. Prove the endpoint responds with a 200 and stop.
- **DoS / concurrency**: a single reproduction ≤ 60s; five serial runs is plenty. Race-condition concurrency 50–100, never 1000+. For rate-limitless SMS / email endpoints, send 5–10 messages to your own phone.
- **Leave nothing behind**: webshells, heapdumps, backups, dumped source — keep locally, delete immediately after reporting; never push to GitHub / third-party drives.
- **Credentials: verify, never use**: leaked AWS / Stripe / database credentials are only verified with `sts get-caller-identity` or a banner; never used for charges, email or connecting to production databases.
- **Redact all PII in reports**: phone numbers, emails, usernames, tokens and cookies keep the first 2 + last 2 characters; attach a sha256 fingerprint where needed to prove you held the original.
- **OOB verification**: don't use public DNSLog platforms; use the vendor's SSRF testing platform, or self-host interactsh / your own DNSLog.
- **No packet, no finding**: every assertion needs an HTTP request / screenshot / video; never submit on an "it should work" hunch.

Each vulnerability class carries finer limits (DoS is the most sensitive, uploads leave no webshell, read classes stop at one sample, etc.) — see the last section of the corresponding playbook.

## Links

[linuxdo](https://linux.do/)

## License

MIT.

All data sources are public material. This project mainly organizes, translates and categorizes that material, packaged as a Claude Code skill for black-box vulnerability hunting.

> 🇨🇳 中文文档: [README.md](README.md)
