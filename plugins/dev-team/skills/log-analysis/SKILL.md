---
name: log-analysis
description: Use when logs, stack traces, or runtime errors need tracing to their source — "analyze these logs", "what does this stack trace mean", a pasted error or crashing-service output. Produces the key error, failure timeline, likely file:line, ranked hypotheses, and next checks. Not for full root-cause investigation and fixing (use debug-issue) or building reproduction steps (use reproduce-bug).
argument-hint: [log or error text]
---

# Log Analysis

Analyze logs, stack traces, and runtime errors to locate the source: $ARGUMENTS

**Use when:**
- Logs, a stack trace, or error output were pasted or pointed to and the failing code needs locating.
- A service fails in CI, staging, or production and log output is the main evidence available.

**Not for:**
- Full investigation through fix and verification → `debug-issue`.
- Turning a vague report into reliable reproduction steps → `reproduce-bug`.

## Workflow

1. Detect the stack from real files first — `package.json` + lockfile, `go.mod`, `next.config.*`, `Dockerfile`, CI workflows — so the trace format reads correctly (Go panics and goroutine dumps, Node/Next.js stack frames, MongoDB driver error codes).
2. Identify the key error: the first originating error, not the last symptom. Strip cascading noise — retries, timeouts caused by the original failure, repeated identical lines, shutdown spam.
3. Build a timeline: order events by timestamp (watch clock skew and interleaved requests/goroutines); mark where normal operation ends and failure begins.
4. Map the key error to source: follow stack frames to file:line; if frames are framework-internal, grep the repo for the literal error message or code to find the throwing/wrapping site. Read those files and confirm the code matches the trace.
5. Correlate with recent change: `git log --oneline -10` and `git diff` on the implicated files if the failure window suggests a regression.
6. Form hypotheses ranked by evidence — each must cite the log line(s) supporting it.
7. List next checks: specific greps, files to read, or log levels/fields to enable that would confirm or eliminate each hypothesis.
8. Record missing context: logs you would need to go further — adjacent services, a wider time window, debug level, request/trace IDs.

## Safety

- Analysis only — no edits, no state-changing commands.
- Read-only commands only (grep, git log, file reads); never restart services, replay traffic, or rotate logs.
- Never reproduce secrets, tokens, or PII from logs in the report — redact values, keep field names.
- If the logs cannot pin the source, say so — never present a guess as the confirmed origin.

## Output format

## Key error
The originating error quoted exactly, plus one line on why surrounding errors are downstream noise.

## Timeline
Ordered events from last-normal to failure, with timestamps where available.

## Likely source
file:line or component, with the stack-frame or grep evidence pointing there.

## Hypotheses
Possible causes ranked by evidence, each citing the supporting log lines.

## Next checks
Specific greps, files, or read-only commands to confirm or eliminate each hypothesis.

## Missing context
Logs or signals not available that would be needed — other services, earlier window, debug level, request/trace IDs.

## Reporting

- Separate what the logs prove (quoted lines, verified file:line) from what is inferred (hypotheses).
- If the source could not be confirmed against repo files, state that under **Missing context** with exactly what is needed.
