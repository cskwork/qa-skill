# qa — five parallel QA agents, one verified report

[![Skill](https://img.shields.io/badge/Claude_Code-Skill-6b5bd6)](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
[![License](https://img.shields.io/badge/license-MIT-blue)](LICENSE)

**Landing page:** https://cskwork.github.io/qa-skill/ · 한국어 요약은 [아래](#한국어-요약)에 있습니다.

One agent testing an app walks one path at a time and stops at the first plausible story. `/qa` splits the work: it launches **five subagents at once**, each owning a different slice of the app, then **reproduces every finding itself** before you see it.

- **`/qa e2e`**: five scenario-based users drive the real app in a browser: the main changed journey, other roles and permissions, edge input, state and interruptions, and regression neighbours.
- **`/qa data`**: five data analysts check the recent changes against real data: API response vs DB rows, query logic, integrity, aggregates and time boundaries, and before-vs-after query results.
- **`/qa all`**: both teams in one launch.

Delegates **find and propose**; they change nothing. You approve fixes from the report, and the coordinator applies them one at a time and re-runs the repro.

## Usage

```text
/qa [e2e|data|all] [target] [--agents N] [--model M] [--effort E] [--since REF]
```

| Arg | Default | Values |
|---|---|---|
| mode | `e2e` | `e2e`, `data`, `all` |
| target | current repo and its dev environment | URL, repo path, ticket key, feature name |
| `--agents N` | `5` | 1-10 per mode |
| `--model M` | `sonnet` (Sonnet 5.5) | `sonnet` `opus` `haiku` `fable` or a full model id; non-Claude ids in `data` mode via a peer CLI |
| `--effort E` | `medium` | `low` `medium` `high` `xhigh` `max` |
| `--since REF` | merge-base with the default branch | git ref or date |

Examples:

```text
/qa e2e https://dev.example.com
/qa data --since v1.4.0
/qa all --model opus --effort high --agents 3
```

## How a run goes

1. **Ground**: read repo instructions and `git diff` since `--since`; list changed endpoints, queries, tables, and screens; write `plan.md`.
2. **Slice**: every changed path maps to a slice; slices share no accounts, data, or browser sessions.
3. **Launch**: one message, N subagents in parallel, each with the same brief template and a written `findings-<n>.md`.
4. **Verify**: merge findings with a shared root cause, reproduce each one, mark it CONFIRMED or UNVERIFIED, clean up test data and browser sessions.
5. **Report**: severity counts, a findings table with evidence and fix proposals, coverage gaps, then one question: which fixes to apply.

## Safety

- Production is out of scope; the DB host is checked before the first query.
- Delegates run read-only queries, leave the repo untouched, and post nothing externally.
- Test data carries a `QA-<run-id>-<n>` tag and is listed for cleanup; secrets are written as `***`.
- Code fixes and data writes happen only after you approve them.

## Smoke run

On a small demo site with login and CRUD (2026-11-09, Sonnet 5.5 medium, five agents, 1.5-2.5 min each): 34 raw findings, merged into **18**, of which **17 were reproduced** by the coordinator. The top one was a blocker: the session check accepted any cookie containing the fixed session string, so a forged cookie read the API without logging in. Three lessons from that run are in the skill: keep one named browser session per agent, delete stress data right after measuring, and note out-of-slice issues in one line instead of chasing them.

## Install

```sh
npx skills@latest add cskwork/qa-skill
# or
git clone https://github.com/cskwork/qa-skill ~/.claude/skills/qa
```

Requirements: a host with parallel subagents (Claude Code `Agent` tool), [`playwright-cli`](https://www.npmjs.com/package/@playwright/cli) for `e2e`, `curl`, and a read-only DB client for `data`.

The skill is user-invoked (`disable-model-invocation: true`): type `/qa`; agents do not start it on their own.

## Files

| Path | What |
|---|---|
| `SKILL.md` | Arguments, safety, the five steps, the delegate brief |
| `docs/index.html` | Landing page (KO/EN) |

Also part of the skill collection [cskwork/skills](https://github.com/cskwork/skills).

## 한국어 요약

`/qa`는 QA 서브에이전트 5개를 동시에 띄우고, 찾은 문제를 직접 재현해 확인한 뒤 보고서 하나로 돌려주는 스킬입니다. `e2e`는 실제 사용자 시나리오로 화면을 조작해 버그를 찾고, `data`는 최근 변경(API·쿼리)을 실제 데이터와 맞춰 봅니다. 기본 모델은 Sonnet 5.5 medium이고 `--model`, `--effort`로 바꿉니다. 서브에이전트는 문제와 수정안만 만들고, 실제 수정은 사용자가 승인한 것만 하나씩 적용합니다. 운영 DB는 쓰지 않습니다.

## License

MIT
