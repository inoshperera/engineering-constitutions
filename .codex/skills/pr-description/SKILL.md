---
name: pr-description
description: Generate a pull request description in a "Purpose / Goals / Approach / Testing" format from the current code changes (git diff/log) or from a plain-language description of what changed. Use this whenever the user asks to write, draft, or generate a PR description, GitHub PR summary, or commit/PR writeup, or asks "can you describe these changes for a PR" — even if they don't name the format explicitly. Always output the result directly in the chat as a copy-pasteable markdown block; never write it to a file unless the user explicitly asks to save it.
---

# PR Description Generator

Write pull request descriptions that explain the *mechanism* of a change — what was actually broken or inconsistent, in terms of real files/functions/behavior — not vague marketing language like "improved performance" or "fixed a bug." A reviewer should finish reading and understand exactly what changed and why, including any non-obvious tradeoffs discovered while implementing it.

## Output rule (important)

Always print the finished description directly in your chat response, inside a single fenced markdown block the user can copy straight into GitHub's PR description box. Do not create or write a file for it unless the user explicitly asks you to save/write it somewhere. The whole point is speed — the user wants to copy-paste, not go find a file.

## Step 1: Gather what actually changed

Don't guess or summarize from memory — ground the description in real change data.

- If you have shell/tool access to the repo, run something like `git diff` (or `git diff --staged`, or `git diff <base>...HEAD` for a branch) and `git log` for recent commit messages. Read enough of the actual diff to understand *why* each change was made, not just which lines moved — file names and line counts alone produce shallow, generic descriptions.
- If earlier in the conversation you (or the user) already made and discussed a set of edits, that conversation context counts as "what changed" too — you don't need to re-derive it from git if you already understand it in detail.
- If you have no shell access and no relevant conversation context, ask the user to paste their diff, or describe the changes in their own words (file by file is fine). Don't fabricate a plausible-sounding diff.

## Step 2: Write the four sections

Use exactly this structure and these headers:

```markdown
## Purpose
<2-5 sentences>

## Goals
<3-5 bullets>

## Approach
<one bullet per meaningfully-changed file or unit>

## Testing
<bullet list>
```

**Purpose** — Explain what was broken, inefficient, or inconsistent *before* this change, from a "why does this exist" angle. Name the actual mechanism: which endpoint gets called redundantly, which field renders inconsistently, which reducer branch does what. Someone who has never seen the code should understand the root cause after reading this, not just "there was a bug."

**Goals** — 3-5 bullets, each one sentence, each starting with a verb describing an outcome (Eliminate, Reduce, Make, Stop, Never, Ensure...). These are the acceptance criteria the Approach section should visibly satisfy — don't list goals the changes don't actually address.

**Approach** — One bullet per meaningfully-changed file, module, or logical unit. Bold the file/module name first, then describe *what* changed and *why* in the same bullet. This is the section most worth spending effort on:
- If something behaved non-obviously and forced a different implementation than the "obvious" fix, say so explicitly — e.g. "X doesn't fire a native DOM event when the value changes programmatically, so `watch()` can't track it reactively; used local state instead." That kind of note is what makes a description useful to a reviewer instead of decorative.
- Skip files that only changed incidentally (formatting-only, import reordering) unless the user asks for exhaustive coverage.
- Order roughly by how central the file is to the change, not alphabetically.

**Testing** — List what was actually verified, and be honest about what wasn't. If you (the assistant) made these changes but never ran them in a real browser/environment/test suite, say that explicitly — e.g. "Not verified in a live browser session; please confirm the dropdown renders correctly in dark mode" — rather than implying confidence you don't have. If the user tells you they tested something themselves (e.g. they pasted a screenshot showing it works), that's legitimate to cite as verified.

## Style notes

- Technical and precise, not promotional. No "this dramatically improves X" — say what changed and let the reader judge the impact.
- Prefer naming real identifiers (function names, field names, component names) over generic phrases like "the logic" or "the handler."
- It's fine — good, even — to call out a real limitation or an edge case you're not fully sure about, rather than glossing over it to sound more finished than the work actually is.
- Match the scope of the four sections to the scope of the change. A one-file bugfix might have a two-sentence Purpose and two Approach bullets; a multi-file refactor earns more detail. Don't pad a small change to look weightier than it is.

## Example

**Input** (paraphrased conversation context): added an in-memory cache in `ExServer` so MQTT reconnects with the same still-valid token skip a redundant `/oauth2/introspect` call, keyed on token with real expiry stored (not just a cached flag), 1-hour TTL + 10k entry cap as safety nets only.

**Output:**

```markdown
## Purpose
The EMQX exhook (`ExServer.onClientAuthenticate`) called the `/oauth2/introspect` endpoint on every MQTT client connect, including reconnects using the exact same token. For devices with flaky connections or frequent reconnects, this meant a redundant network round-trip and unnecessary load on the key manager for every single connection attempt, even when the token's validity hadn't changed.

## Goals
- Eliminate redundant introspection calls for a token that was already validated recently.
- Reduce connect latency for devices reconnecting with a still-valid token.
- Never treat a token as valid past its real, server-issued expiry — caching must not weaken correctness.

## Approach
- **`ExServer`**: Added an in-memory `Cache<String, CachedTokenInfo>` (Guava `CacheBuilder`), keyed by the same access token used elsewhere in the class. Each entry stores the token's scope plus its real expiry (`exp` from the introspection response), not just an opaque valid/invalid flag, so cache hits can still be correctly rejected once the token has actually expired. On `authenticate`: check the cache first; if cached and not actually expired, reuse it and skip introspection; if genuinely expired, invalidate and re-introspect; on a fresh introspection success, cache the result.
- **`HandlerConstants`**: Cache uses a 1-hour TTL (`TOKEN_CACHE_TTL_SECONDS`) plus a 10,000-entry size cap as safety nets only — the real `exp` check is what actually governs correctness, so these bounds exist purely to cap memory, not to enforce validity.

## Testing
- Verified via unit test that a cached, non-expired token skips the introspection call on a second `authenticate` invocation.
- Verified that a cached token past its real `exp` is invalidated and triggers re-introspection rather than being served stale.
- Not load-tested against production-scale reconnect volume; the entry cap (10k) is a guess based on expected device fleet size and may need tuning.
```
