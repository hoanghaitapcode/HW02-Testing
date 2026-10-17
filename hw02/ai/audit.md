# AI audit - current preparation step

Status: partial working record, not the complete mandatory HW02 AI Audit appendix.

Declaration: I use AI tools for the following tasks: local project inspection and drafting a requirements/domain/boundary analysis for mobile FR-04.

## Interaction recorded

- Tool: Codex desktop assistant. Exact model identifier was not independently verified.
- Date: 2026-10-17 (Asia/Bangkok). The setup inspection timestamp was approximately 18:01; individual chat message timestamps were not independently captured.
- User prompt, verbatim: `do 1 and 2`
- Context of requested steps:
  1. Inspect the local project and requirements to confirm setup and testing scope.
  2. Create Markdown documentation for requirements, valid/invalid domains, boundaries, and assumptions, starting with mobile FR-04.
- Inputs: local README, API specification, setup guide, package manifests, relevant mobile/backend source, read-only Git inspection, and three HTTP reachability checks.
- Output artifacts: `hw02/README.md`, `hw02/reports/setup-inspection.md`, `hw02/reports/FR04-mobile-profile-analysis.md`, and this audit record.
- Outputs are preserved as the generated Markdown files. Save the full chat transcript and tool outputs as supporting material; this summary is not a verbatim replacement for them.
- Human review: pending. Student must record corrections and reasons after review.

## Review points

- AI identified ambiguous empty-field, normalization, and name/address length rules rather than making up constraints.
- AI distinguished source-review concerns from executed test results.
- Initial Git inspection encountered an ownership warning; subsequent inspection used a per-command trust exception without changing global configuration.
- Existing database changes and dependency folders were preserved.
- No feature tests, installations, fixes, commits, or GitHub posts were performed.

## Audit completion still required

Earlier interactions in this chat (assignment interpretation, feature selection, setup advice) also used AI. Add their actual prompts, outputs, tool name, and recorded date/time from available chat records. Do not invent unavailable timestamps. Retain the complete transcript for the current step as well. Add later interactions as work continues.

The mandatory 200-300-word personal AI Critique has not been written. It should reflect the student's actual review and testing experience.

## Interaction - FR-03 design and commit checkpoint

- Tool: Codex desktop assistant; exact model identifier not independently verified.
- Date: 2026-10-17 (Asia/Bangkok); exact per-message time is available from chat records if retained, not independently captured here.
- User prompt, verbatim: `now write domain testing for 4 features but i you will write for each features, after 1, i need to commit onto github to get commit log`
- Interpreted scope for this turn: draft FR-03 first, including Domain Testing and BVA, then stop for student review and commit. Other feature reports wait for the student's checkpoint.
- Inputs: local business requirements, API documentation, web reset component, relevant backend handlers, existing preparation documents, and Git status.
- Output: `hw02/reports/FR03-web-password-reset.md`, plus updates to `hw02/README.md` and this log.
- Work: requirement predicates, domains, on/off/in sample selection, single-rule violations, interactions, boundary samples, test procedures, expected outcomes, ambiguities, and source concerns.
- Execution: no feature tests run; actual results remain unrecorded. Parameterized cases are identified separately from row counts.
- Human review: pending; student should record corrections and the reason for each.
- Commit and push: left to the student as requested. No GitHub issue was posted.
- Retain the full chat and generated report as the original AI output; this entry is a summary, not a complete transcript.
