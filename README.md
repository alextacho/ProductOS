# Workbench

A collection of opinionated AI tools for product people — each targeting a specific PM job.

Each tool is self-contained: a clear input, a designed process, a useful output.

## Tools

| Tool | Status | Job |
|------|--------|-----|
| [Competitive Analysis](./competitive-analysis/) | In development | Understand the competitive landscape for a product or market |

## Underlying System

Several workbench tools share the same underlying pattern: track a set of entities, collect signals, detect change, synthesize meaning, deliver insight. Competitors, customers, markets — same system, different domain layer.

See [`SYSTEM.md`](./SYSTEM.md) for the two-layer model and the design principles that keep system and domain cleanly separated as the workbench grows.

## Principles

- **Job-specific, not general.** Each tool does one thing well. No prompt playgrounds.
- **Output-first.** Every tool is designed backward from the deliverable users actually need.
- **Craft embedded.** The PM judgment — what questions to ask, what to look for, how to frame it — is built in.