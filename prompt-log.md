# Prompt Log

## 2026-10-09
- **What I asked:** Requested initial setup for portfolio repository `firstname-lastname` with standard directory skeleton, baseline `AGENTS.md` tailored to my resume, `CLAUDE.md`, `.gitignore`, `RESUME.md`, and first log entry.
- **What you produced:** Generated full repository layout, `.gitignore`, canonical rule pointers, tailored `AGENTS.md`, `RESUME.md`, and initial `prompt-log.md` entry.
- **What was wrong and how it was caught:** N/A (initial repository setup session created directly from specification).

## 2026-11-12
**Tool:** Claude (claude.ai)

- **What I asked:** I pasted the Stage 2 feedback and the link to my spec, and asked Claude to help me fix each gap. First I asked Claude to open my current spec. Then I asked for help identifying where the changes needed to be made. I then gave Claude the scenario 36 call quote (strike at spot, premium USD 0.0296 per EUR) to fill the placeholders.
- **What you produced:** A plan in the feedback's order: move the spec, add the integrity rules, create the README, add a Stage 4 source column, add the call, add the tab architecture, and run a review. A revised `spec.md` (version 0.2). Call figures from the quote: premium $65,120, future value about $66,496, and a call cap of about −$2,531,376.
- **What was wrong and how it was caught:** The call figures were left as placeholders** because the quote was not in the files I had shared, and Claude did not guess them. After I supplied the quote, Claude ran the arithmetic to check the call figures (DF_USD = 1.02113, premium future value $66,496.13, cap −$2,531,376.13) and searched the files to confirm no placeholders were left.
