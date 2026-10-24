---
name: pending-write-review
description: "Review staged Hermes skill and memory writes."
version: 0.1.0
author: "Arden (Ardenye), Hermes Agent"
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [Hermes, Skills, Memory, Approvals, Review]
    category: autonomous-ai-agents
    related_skills: [hermes-agent, hermes-memory-maintenance]
---

# Pending Write Review Skill

A careful workflow for reviewing staged Hermes Skill and Memory writes in bounded batches. It helps the agent assess each proposal and verify authorized outcomes; it does not bypass Hermes approval gates or replace the user's decision.

## When to Use

- The user asks to inspect or process pending Skill or Memory writes.
- A task already in progress includes specific pending IDs that need review.
- Do not inspect or process queues proactively merely because they may exist.

## Prerequisites

- Access to the relevant profile's Hermes Pending review commands and read-only file inspection tools.
- The active Hermes home for each profile in scope; profiles have separate queues and memory.
- No external dependencies. Use the current Hermes CLI/chat review surface and native tools such as `terminal`, `read_file`, `search_files`, and `skill_view`.

## How to Run

Treat “review” and “process” as different requests. Review-only means no approvals or rejections. A request to process a queue starts the review and manifest workflow; it is not, by itself, authorization to apply a proposed batch. Before any write, obtain explicit authorization for the exact manifest.

## Quick Reference

| Stage | Required result |
|---|---|
| Snapshot | Live per-profile IDs and counts; late arrivals kept outside scope |
| Review | Full payload and exact target checked for each ID |
| Manifest | One row per ID with action, target, disposition, and reason |
| Authorization | Explicit confirmation of that manifest before any write |
| Execution | Official Hermes handlers only; no direct Pending-file edits |
| Verification | Read back Pending state and approved destinations |

## Procedure

### 1. Freeze a live, bounded snapshot

1. Resolve the active Hermes home for every profile in scope from live runtime state. Do not infer another profile's path from a previous session.
2. List the live Skill and Memory Pending records separately for each profile. Record exact IDs and counts; treat this as the batch boundary.
3. Include records present when the snapshot is taken. Keep arrivals after the snapshot outside the manifest and report them separately.
4. Never edit or delete Pending JSON directly. Never disable an approval gate to make a batch easier to process.

### 2. Review each complete candidate

For every ID in the snapshot:

- Read the full payload, not only its summary. Record profile, subsystem, operation type, target carrier or Skill, and all operations in a batch.
- Compare it with the exact current target and other pending changes to that target. Check correctness, profile scope, duplication, conflicts, durability, sensitivity, and whether the proposed carrier is appropriate.
- For Skill changes, use the currently supported Hermes review/diff interface and inspect every operation. A clean diff proves applicability, not correctness or safety. Never approve a batch with a failed operation as submitted.
- For Memory changes, inspect the exact target entry and verify replacement scope. Check the current MemoryStore semantics and character budget; do not assume that an old-text locator changes only that span. If a supported Memory diff command is unavailable, compare the complete Pending payload with the current carrier using read-only tools. Do not invoke internal write appliers as a substitute for review.
- Verify technical claims against current Hermes docs, CLI help, or relevant source when needed. Label claims that remain unverified.
- Do not expose raw sensitive memory content in the batch summary. Use concise categories and exact IDs instead.

### 3. Assign a disposition

Follow the user's latest explicit rules for this bounded snapshot. For each candidate, recommend one of:

- **Approve** — fully reviewed, technically sound, applicable, non-duplicative, and safe to apply.
- **Reject** — failed anchors or content the user has directed to reject and that cannot safely be applied as submitted.
- **Defer** — unresolved facts or security questions, incomplete evidence, conflicting alternatives needing consolidation, or an explicit hold not superseded by a later instruction.

Do not convert a previous batch instruction into standing authorization for future runs. Do not treat an over-budget proposal as safe to apply; report the exact projected limit and defer or reject according to the user's explicit policy. Do not delete unrelated entries to make room unless that consolidation is separately included in the authorized manifest.

### 4. Prepare one exact manifest and wait

Before the first write, show one concise manifest with a row for every ID:

- Profile and subsystem.
- Exact ID, target carrier or Skill, and operation type.
- Proposed disposition and a brief, non-sensitive reason.
- The total counts for approve, reject, and defer.

Before proposing rejection, inspect the current handler and state whether it archives or permanently removes the Pending record. Do not assume a reject is reversible, and do not create a backup unless separately authorized.

Wait for explicit authorization of that exact manifest. One clear batch confirmation is enough; do not ask for separate per-ID confirmations. If the user changes the scope, a new ID appears, or the proposed payload changes, present the delta and obtain authorization for that delta before acting.

### 5. Execute only the authorized operations

- Use the supported Hermes review commands/handlers for the specified profile and subsystem. Verify the current command surface rather than assuming an internal Python API is public or stable.
- Process only IDs and dispositions in the confirmed manifest. Never use an “approve all” or “reject all” form if it could include IDs outside the manifest.
- Track every handler result independently. Batches may partially succeed; a failure for one ID does not roll back successful operations for others.
- Do not stage or re-stage a candidate while processing unless that separate write is explicitly authorized. A changed or newly generated Pending ID is outside the old manifest.
- Do not commit, push, publish, sync, or change configuration as part of queue processing unless separately authorized.

### 6. Verify and report

Re-list every affected profile/subsystem after processing. Verify that:

- Each approved ID is absent from Pending and its exact target contains the intended change.
- Each rejected ID is absent from Pending; describe it as removed or archived only if the handler's behavior confirms that outcome.
- Deferred IDs and out-of-snapshot arrivals remain pending.
- Per-profile counts and remaining IDs match the read-back.

Report exact approved, rejected, and deferred IDs; any handler failures or warnings; remaining counts; and any target that could not be verified. Never claim the queue is clear while a record remains unreviewed or outside the authorized scope.

## Pitfalls

- Approval settings and Pending queues are profile-scoped; a count from one profile is not a global count.
- A staged write is not an applied write. Approval gates do not guarantee that a candidate is accurate, non-duplicative, or safe.
- Do not assume the current rejection implementation is permanent or reversible. Inspect it for the active Hermes version before telling the user what will happen.
- Do not copy raw memory entries into chat or a manifest when a concise risk category is enough.
- Do not infer Git or external-memory synchronization from a local Skill/Memory change.

## Verification

- Every ID in the authorized manifest has an individually recorded handler result.
- The live queue has been read back for each affected profile and matches the expected remaining IDs.
- Every approved destination was read back; no unapproved ID or target was changed.
- The final report distinguishes completed operations, deferred records, late arrivals, and failures.
