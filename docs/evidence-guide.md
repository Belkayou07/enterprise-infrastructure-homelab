# Evidence Capture Guide

I use screenshots as supporting evidence for my BelkaCorp homelab. They do not replace the written documentation; they prove that I actually implemented, observed, verified, or troubleshot the state described in the repository.

## Evidence Rule

I capture evidence at meaningful checkpoints rather than screenshotting every command.

I prefer evidence that proves one of these things:

- a design or configuration was actually implemented;
- the final state was verified;
- a real problem was observed;
- a troubleshooting change resolved the problem;
- a GUI view adds useful visual context to a PowerShell or terminal verification.

## What Makes a Good Screenshot

A useful screenshot should:

- show the relevant command and result or the relevant management-console state;
- include enough context for a reviewer to understand what is being proven;
- avoid passwords, product keys, tokens, personal email addresses, and other secrets;
- avoid unrelated desktop clutter when practical;
- have one clear purpose;
- use the project naming convention once committed to GitHub.

## Naming Convention

Committed evidence files use:

```text
chapter-step-short-description.png
```

Examples from the real project:

```text
00-03-smt-24-logical-processors.png
01-01-hyperv-verification.png
01-03b-virtual-switches-final.png
01-06d-test-vm-bidirectional-ping.png
03-05-fw01-initial-preboot-audit.png
```

Suffixes such as `a`, `b`, `c`, and `d` are useful when several screenshots belong to the same technical step and tell a sequence such as before-state, remediation, and final verification.

## Evidence Levels

### Design Evidence

I use diagrams, schemas, addressing plans, and documented engineering decisions when they prove design intent better than a screenshot.

### Implementation Evidence

I capture configuration screens or commands when they show that I actually created or changed a resource.

### Verification Evidence

I capture commands or management consoles that prove the intended final state exists.

### Troubleshooting Evidence

When a real problem occurs, I keep useful before/after evidence when it helps explain:

```text
SYMPTOM
   |
   v
INVESTIGATION
   |
   v
ROOT CAUSE
   |
   v
REMEDIATION
   |
   v
FINAL VERIFICATION
```

I do not recreate a historical failure artificially just to make the portfolio look better.

## Evidence Status

The source of truth is [`../screenshots/evidence-index.md`](../screenshots/evidence-index.md).

I use two statuses:

- `CAPTURED` — I took the screenshot, but the actual image file is not yet stored under its intended repository path.
- `COMMITTED` — the actual evidence file exists in GitHub and can be opened from the evidence index.

I only mark evidence `COMMITTED` after the file is really present in the repository.

## Presentation Convention

To keep the repository visually consistent:

- chapter documentation such as `01-hyper-v-platform.md`, `02-network-architecture.md`, and `03-firewall-routing.md` embeds relevant screenshots directly with Markdown image syntax under an `Evidence` heading;
- `screenshots/evidence-index.md` remains the compact evidence catalog and uses links rather than embedding every image;
- screenshot-folder README files track evidence state and may use links because their purpose is inventory rather than narrative documentation.

This keeps the main chapter documents readable as a complete technical walkthrough while preventing the global evidence index from becoming unnecessarily large.

## Screenshot Workflow

I finish evidence and documentation at the same meaningful checkpoint as the technical work. I do not postpone a batch of screenshots or documentation until the end of a chapter.

```text
UNDERSTAND
    |
    v
IMPLEMENT
    |
    v
VERIFY
    |
    v
CAPTURE USEFUL EVIDENCE
    |
    v
STORE + NAME IT CONSISTENTLY
    |
    v
VERIFY THE FILE IN GITHUB
    |
    v
EMBED IT IN THE CHAPTER DOCUMENT WHEN RELEVANT
    |
    v
DOCUMENT THE RESULT
    |
    v
CONTINUE TO THE NEXT STEP
```

I may use both CLI and GUI evidence when they prove different things. PowerShell or terminal output gives me precise, scriptable state; Hyper-V Manager and other management consoles can provide useful visual confirmation.

## Portfolio Principle

A reviewer should be able to follow this chain through the repository:

```text
DESIGN
  |
  v
IMPLEMENT
  |
  v
VERIFY
  |
  v
TROUBLESHOOT WHEN NEEDED
  |
  v
EVIDENCE
  |
  v
EXPLAIN
```

I keep the evidence concise, credible, and tied to results I actually observed in the lab.