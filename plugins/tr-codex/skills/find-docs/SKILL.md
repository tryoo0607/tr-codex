---
name: find-docs
description: >-
  Retrieves up-to-date documentation, API references, and code examples for any
  developer technology. Use this skill whenever the user asks about a specific
  library, framework, SDK, CLI tool, or cloud service — even for well-known ones
  like React, Next.js, Prisma, Express, Tailwind, Django, or Spring Boot. Your
  training data may not reflect recent API changes or version updates.

  Always use for: API syntax questions, configuration options, version migration
  issues, "how do I" questions mentioning a library name, debugging that involves
  library-specific behavior, setup instructions, and CLI tool usage.

  Use even when you think you know the answer — do not rely on training data
  for API details, signatures, or configuration options as they are frequently
  outdated. Always verify against current docs. Prefer this over web search for
  library documentation and API details.
---

## 이 배포본의 연결

Context7은 필요한 라이브러리 문서 근거를 반환한다. BMAD의 전체 조사나 작업 상태를 대신 관리하지 않는다. 설치된 `ctx7 --version`을 [도구 manifest](../../scripts/dev-tools.toml)와 대조하고, 부재나 불일치는 설치 담당으로 돌려준다. 일반 조회 중 설치나 업데이트, 로그인, MCP 등록을 실행하지 않는다. 필요한 문서만 읽고 라이브러리 ID와 버전, 출처 및 확인하지 못한 범위를 반환한다. 이미 확보한 정확한 버전의 근거는 재사용한다. 조회 문자열에는 내부 코드나 비밀을 넣지 않는다.


# Documentation Lookup

Retrieve current documentation and code examples for any library using the Context7 CLI.

Use the installed, version-pinned `ctx7` CLI:

```bash
ctx7 library <name> "<query>"
ctx7 docs <libraryId> "<query>"
```

Installation and updates belong to toolchain setup, outside this lookup workflow.

## Workflow

Two-step process: resolve the library name to an ID, then query docs with that ID.

```bash
# Step 1: Resolve library ID
ctx7 library <name> "<query>"

# Step 2: Query documentation
ctx7 docs <libraryId> "<query>"
```

You MUST call `library` first to obtain a valid library ID UNLESS the user explicitly provides a library ID in the format `/org/project` or `/org/project/version`.

IMPORTANT: Do not run these commands more than 3 times per question. If three attempts do not provide sufficient evidence, report that limit and check official documentation or source instead of treating the best partial result as complete.

## Step 1: Resolve a Library

Resolves a package/product name to a Context7-compatible library ID and returns matching libraries.

```bash
ctx7 library React "How to clean up useEffect with async operations"
ctx7 library "Next.js" "How to set up app router with a proxy"
ctx7 library Prisma "How to define one-to-many relations with cascade delete"
```

Use the official library name with proper punctuation (e.g., "Next.js" not "nextjs", "Customer.io" not "customerio", "Three.js" not "threejs"). If results look wrong, try alternate spellings such as `next.js` before changing the query.

Always pass a `query` argument — it is required and directly affects result ranking. Use the user's intent to form the query, which helps disambiguate when multiple libraries share a similar name. Do not include any sensitive or confidential information such as API keys, passwords, credentials, personal data, or proprietary code in your query.

### Result fields

Each result includes:

- **Library ID** — Context7-compatible identifier (format: `/org/project`)
- **Name** — Library or package name
- **Description** — Short summary
- **Code Snippets** — Number of available code examples
- **Source Reputation** — Authority indicator (High, Medium, Low, or Unknown)
- **Benchmark Score** — Quality indicator (100 is the highest score)
- **Versions** — List of versions if available. Use one of those versions if the user provides a version in their query. The format is `/org/project/version`.

### Selection process

1. Analyze the query to understand what library/package the user is looking for
2. Select the most relevant match based on:
   - Name similarity to the query (exact matches prioritized)
   - Description relevance to the query's intent
   - Documentation coverage (prioritize libraries with higher Code Snippet counts)
   - Source reputation (consider libraries with High or Medium reputation more authoritative)
   - Benchmark score (higher is better, 100 is the maximum)
3. If multiple good matches exist, acknowledge this but proceed with the most relevant one
4. If no good matches exist, clearly state this and suggest query refinements
5. For ambiguous queries, request clarification before proceeding with a best-guess match

### Version-specific IDs

If the user mentions a specific version, use a version-specific library ID:

```bash
# General (latest indexed)
ctx7 docs /vercel/next.js "How to set up app router"

# Version-specific
ctx7 docs /vercel/next.js/v16.2.9 "How to set up app router"
```

The available versions are listed in the `library` output. Use the requested version when indexed. If absent, explicitly report the mismatch and verify the exact version in its official documentation or source; never silently substitute a nearby version.

## Step 2: Query Documentation

Retrieves up-to-date documentation and code examples for the resolved library.

```bash
ctx7 docs /facebook/react "How to clean up useEffect with async operations"
ctx7 docs /vercel/next.js "How to add an authentication proxy to app router"
ctx7 docs /prisma/prisma "How to define one-to-many relations with cascade delete"
```

If you know the programming language the user writes in, add `--language` so code examples in that language rank first:

```bash
ctx7 docs /websites/twilio "How to send an SMS" --language Ruby
```

### Writing good queries

The query directly affects the quality of results. Be specific and include relevant details, but keep each query to one topic — if the question spans multiple distinct concepts, run a separate `docs` command per concept instead of combining them, unless the question is about how the concepts interact. Do not include any sensitive or confidential information such as API keys, passwords, credentials, personal data, or proprietary code in your query.

| Quality         | Example                                                    |
| --------------- | ---------------------------------------------------------- |
| Good            | `"How to set up authentication with JWT in Express.js"`    |
| Good            | `"React useEffect cleanup function with async operations"` |
| Bad (too vague) | `"auth"`                                                   |
| Bad (too vague) | `"hooks"`                                                  |
| Bad (too broad) | `"routing and auth and caching in Next.js"`                |

Describe what to look up in the library's documentation, rather than the task to complete — vague one-word queries return generic results, and multi-topic queries dilute ranking and return shallow results for each topic.

The output contains two types of content: **code snippets** (titled, with language-tagged blocks) and **info snippets** (prose explanations with breadcrumb context).

## Authentication

Works without authentication. For higher rate limits:

```bash
# Option A: environment variable
export CONTEXT7_API_KEY=your_key

# Option B: OAuth login
ctx7 login
```

## Error Handling

If a command fails with a quota error ("Monthly quota reached" or "quota exceeded"):

1. Inform the user their Context7 quota is exhausted
2. Suggest they authenticate for higher limits: `ctx7 login`
3. If authentication is unavailable or declined, use official documentation or source through available browsing tools. Clearly mark what could not be verified; do not present recalled API details as verified.

Do not silently fall back to training data — always tell the user why Context7 was not used.

## Common Mistakes

- Library IDs require a `/` prefix — `/facebook/react` not `facebook/react`
- Always run `ctx7 library` first — `ctx7 docs react "hooks"` will fail without a valid ID
- Use descriptive queries, not single words — `"React useEffect cleanup function"` not `"hooks"`
- One topic per query — split `"routing and auth and caching"` into a separate `docs` command per concept, unless the question is about how they interact
- Do not include sensitive information (API keys, passwords, credentials) in queries
