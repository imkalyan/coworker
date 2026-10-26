# AI-Powered Code Review: An Agentic Engineering Workflow

> Inspired by Anthropic's published Claude Code code-review practices.
>
> Automate routine code review, reserve human attention for high-risk decisions, and maintain software quality as AI-generated code scales.

## Table of Contents

- [Overview](#overview)
- [Core Principles](#core-principles)
- [Architecture](#architecture)
- [Code Review Lifecycle](#code-review-lifecycle)
- [AI Review Engine](#ai-review-engine)
- [Risk-Based Approval](#risk-based-approval)
- [Security and CI Enforcement](#security-and-ci-enforcement)
- [Feedback and Rollback](#feedback-and-rollback)
- [Continuous Evaluation](#continuous-evaluation)
- [Repository Configuration](#repository-configuration)
- [Implementation Roadmap](#implementation-roadmap)
- [Metrics and Observability](#metrics-and-observability)
- [Best Practices](#best-practices)
- [Future Enhancements](#future-enhancements)

## Overview

AI coding agents increase the volume and speed of software development. Traditional code review—where engineers manually inspect every line—becomes difficult to scale as pull-request volume grows.

The answer is not to eliminate code review. It is to redesign it.

An effective AI-powered review system should:

1. Detect bugs before they ship.
2. Verify changes through executable checks.
3. Identify security vulnerabilities and regressions.
4. Route high-risk changes to the right human reviewers.
5. Reduce false positives and unnecessary review cycles.
6. Make mistakes fast and safe to roll back.
7. Learn from incidents, reverts, and historical review outcomes.

**Objective:** minimize human effort per change without increasing production incidents or regressions.

These are design goals, not guaranteed outcomes. Validate them against your team's baseline.

## Core Principles

### 1. Automate verification, not accountability

Agents can inspect code, run tests, investigate defects, and propose fixes. The organization must retain clear ownership of the code it ships.

### 2. Review continuously

Review should start while code is being developed. The agent should verify its implementation, inspect its own changes, and correct routine defects before asking for human attention.

### 3. Apply review rigor according to risk

A documentation change and an authentication change should not require the same scrutiny. Review depth should reflect business impact, security implications, system criticality, complexity, detectability, and recovery cost.

### 4. Optimize total engineering effort

Faster reviews are not a win if they create more incidents. Measure review effort and downstream defect cost together.

### 5. Make failures reversible

Assume some defects will escape review. Design deployment and release processes so they can be detected and reversed quickly.

### 6. Build a learning system

Use incidents, reverts, and validated findings to improve tests, rules, context, tooling, and evaluation datasets.

## Architecture

The following is a proposed architecture based on the principles in the source article; it is not a claim to reproduce Anthropic's exact internal implementation.

```text
                 Developer / Coding Agent
                           |
                           v
                  Local Verification
             - Build and compilation
             - Unit and integration tests
             - Lint and formatting
             - Diff inspection
                           |
                           v
                      Pull Request
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
        AI Code Review  Security Review  CI Pipeline
             |             |             |
             v             v             v
        Bug Findings  Security Findings  Test Results
             |             |             |
             +-------------+-------------+
                           |
                           v
                  Findings Validation
             - Deduplication
             - False-positive filtering
             - Severity classification
             - Evidence validation
                           |
                           v
                   Risk Classification
                           |
                           v
                  Human Approval Policy
                           |
                           v
                      Merge Queue
                           |
                           v
                   Staged Deployment
                           |
                           v
                  Production Monitoring
                           |
                +----------+----------+
                |                     |
                v                     v
            Healthy Release     Incident / Regression
                                      |
                                      v
                                Rollback / Revert
                                      |
                                      v
                              Evaluation Dataset
                                      |
                                      v
                               Improve Review
```

| Component | Responsibility |
|---|---|
| Local verification | Catch implementation errors before PR creation. |
| AI code reviewer | Identify defects introduced by the change. |
| Security reviewer | Detect security-sensitive problems. |
| CI pipeline | Execute deterministic quality and correctness checks. |
| Findings validator | Reduce duplicate and unsupported comments. |
| Risk classifier | Assign review requirements based on change risk. |
| Approval controller | Enforce human approval and repository policies. |
| Merge queue | Verify integration with the latest target branch. |
| Deployment controller | Support staged releases and feature flags. |
| Rollback mechanism | Reverse problematic changes quickly. |
| Evaluation engine | Measure review quality against known defects and clean changes. |

**Important:** AI review is probabilistic. CI checks and repository protections should enforce deterministic requirements wherever possible.

## Code Review Lifecycle

### 1. Pre-PR verification

Before opening a pull request, the coding agent should:

- Compile the affected modules.
- Run relevant unit and integration tests.
- Execute formatters and linters.
- Inspect the complete diff.
- Verify the original requirement.
- Check for unintended changes and edge cases.
- Validate UI changes where applicable.
- Report checks that could not be completed.

Require evidence of execution rather than accepting a statement that the task is complete.

Example instruction:

```text
Implement the requested change.

Before finishing:
1. Run the relevant tests.
2. Build the affected modules.
3. Inspect the complete Git diff.
4. Look for regressions and unintended changes.
5. Report commands executed and their results.
6. Identify checks that could not be completed.

Do not claim a check passed unless you have evidence
that it actually ran successfully.
```

#### Fresh-context review

After implementation, use a separate reviewer with a clean context to inspect the change. Provide the original requirement, expected behavior, affected modules, relevant conventions, and full diff.

### 2. AI review on every PR

The reviewer should inspect the change in the context of the existing system, not just the changed lines.

Review dimensions:

- **Correctness:** Does the implementation satisfy the requirement?
- **Regression risk:** Could existing behavior break?
- **Data integrity:** Could data be lost, duplicated, corrupted, or misrouted?
- **Concurrency:** Could race conditions or ordering problems appear?
- **Error handling:** Are failures handled correctly?
- **Performance:** Could latency or resource consumption regress?
- **Security:** Could sensitive data or access controls be exposed?
- **Maintainability:** Are important architectural constraints violated?

Example reviewer prompt:

```text
You are a senior software engineer reviewing a pull request.

Your primary objective is to identify real defects introduced
by the proposed change.

Review the diff against:
- The pull request's stated intent.
- Existing implementation and call sites.
- Relevant architectural conventions.
- Tests and error-handling behavior.
- Security and data-integrity requirements.

For each finding:
1. Explain the defect.
2. Identify the exact file and changed line.
3. Describe a concrete failure scenario.
4. Explain the likely impact.
5. Suggest a minimal correction.

Do not report:
- Purely stylistic preferences.
- Speculative issues without a credible failure scenario.
- Existing defects unrelated to this PR.
- Findings already addressed by another comment.

If no actionable issues are found, say so explicitly.
Do not claim tests or commands passed unless execution
results are available.
```

This prompt is a starting point, not a substitute for evaluation against real defects.

### 3. Validate findings

A single AI reviewer can produce incorrect or redundant findings. For higher-confidence review, use independent validation or multiple specialized reviewers.

Suggested workflow:

1. Generate candidate findings.
2. Validate each against code and surrounding context.
3. Check whether the PR introduced the issue.
4. Deduplicate equivalent findings.
5. Assign severity.
6. Publish actionable findings only.

Example structured finding:

```json
{
  "rule_id": "CORRECTNESS-001",
  "severity": "high",
  "category": "correctness",
  "file": "src/PaymentService.java",
  "line": 142,
  "introduced_by_change": true,
  "title": "Missing validation before processing",
  "description": "The modified execution path accepts invalid input.",
  "failure_scenario": "An invalid request reaches the persistence layer.",
  "suggested_fix": "Validate the input before performing the operation."
}
```

The example is illustrative. Do not treat model-generated confidence scores as calibrated probabilities unless empirically validated.

### 4. Security review

Security review should be independent of general code review. Depending on the application, checks may include:

- Authentication and authorization.
- Secrets and credential exposure.
- Input validation and injection.
- Sensitive-data logging.
- Insecure configuration.
- Dependency vulnerabilities.
- Unsafe deserialization.
- File and network access.
- Access-control regressions.

Mandatory security findings should block merging until fixed or explicitly dismissed through an auditable process.

**Do not rely on an LLM alone for security enforcement.** Combine AI analysis with static analysis, dependency scanning, secret detection, and deterministic policy checks.

### 5. Automated CI checks

Enforce checks such as:

- Compilation and build validation.
- Unit and integration tests.
- Type checking, linting, and formatting.
- Static analysis.
- Dependency and security scans.
- API or contract tests.
- Relevant end-to-end tests.

A successful AI review must not override a failed required CI check. Passing CI does not prove that the change is defect-free.

### 6. Risk-based human approval

| Risk level | Example | Illustrative review requirements |
|---|---|---|
| Low | Documentation or isolated nonfunctional changes | Automated review and required CI |
| Medium | Routine business logic | Automated review, CI, and policy-defined human approval |
| High | Transaction processing, data migrations, complex concurrency | Thorough automated review and independent human approval |
| Critical | Authentication, billing, release infrastructure, sensitive security controls | Designated human approval and specialized checks |

The real policy must reflect organizational, regulatory, and operational requirements.

Risk classification can consider changed paths, ownership metadata, complexity, security-sensitive dependencies, schema changes, public API changes, deployment configuration, and potential blast radius.

Mandatory repository rules must take precedence over model-generated risk scores.

### 7. Merge and deployment

After required checks and approvals pass:

1. Add the PR to a merge queue.
2. Validate it against the current target branch.
3. Merge only when required checks succeed.
4. Use feature flags where appropriate.
5. Release to internal users first when practical.
6. Expand deployment gradually.
7. Monitor for regressions.

A merge queue helps prevent individually valid PRs from combining into a broken main branch. Staged deployment limits the impact of defects that escape pre-merge checks.

### 8. Incident detection and rollback

Monitor health checks, error rates, latency, business-level correctness, and alerts. Maintain feature-flag disablement and rapid revert or rollback procedures.

For each incident, ask:

1. What failed?
2. Which change introduced the failure?
3. Why did existing checks miss it?
4. Could a test, rule, or review improvement have caught it?
5. Was detection or rollback slower than necessary?
6. How will the evaluation suite guard against recurrence?

The objective is to improve the system, not merely assign blame.

## AI Review Engine

### Reviewer specialization

| Reviewer | Focus |
|---|---|
| Correctness | Logic errors and incorrect behavior |
| Regression | Interactions with existing functionality |
| Concurrency | Race conditions, ordering, synchronization |
| Security | Vulnerabilities and unsafe access |
| Performance | Latency, throughput, memory, resources |
| Architecture | Dependency boundaries and design constraints |

Not every PR needs every reviewer. Select reviewers according to risk and affected components; indiscriminate parallel review increases cost and duplication.

### Prevent unbounded review loops

Coding and reviewing agents can repeatedly modify and reject the same code. Set explicit limits:

- Limit automated review-fix cycles.
- Re-review the new diff after the initial full review.
- Group related defects into one correction pass.
- Avoid repeating findings already addressed.
- Require a different approach when repeated attempts fail.
- Escalate unresolved high-risk issues to a human.

Example policy:

```text
Initial review: Full relevant diff and context.
Fix cycle 1: Address validated findings.
Fix cycle 2: Re-review changed code and affected paths.
Fix cycle 3: Reassess the implementation approach.

If the change still fails review:
Stop automatic iteration and request human intervention.
```

Three cycles is an example, not a universal optimum.

### Keep comments actionable

Every finding should include an exact file and line, a concrete failure scenario, the impact, whether the PR introduced it, and a suggested fix where practical. Avoid purely stylistic comments that create needless agent commits and review rounds.

## Risk-Based Approval

Use deterministic rules alongside classification.

```text
Pull Request
     |
     v
Inspect changed files
     |
     v
Apply mandatory path rules
     |
     v
Evaluate change impact
     |
     v
Assign risk tier
     |
     +---- Low ------> Automated checks + permitted approval policy
     |
     +---- Medium ---> Automated checks + required review
     |
     +---- High -----> Specialized checks + independent review
     |
     +---- Critical -> Designated human approval
```

Policy precedence should be:

1. Organization and regulatory requirements.
2. Mandatory repository protection rules.
3. Sensitive-path ownership rules.
4. Risk-tier requirements.
5. Optional automated approval for explicitly eligible changes.

A model-generated score must never override mandatory approval requirements. Start conservatively and relax policies only when evidence supports doing so.

## Security and CI Enforcement

A PR is eligible to merge only when required CI passes, mandatory findings are resolved, required human approvals are present, ownership rules are satisfied, and merge-queue requirements are met.

Illustrative pseudocode:

```text
IF required_ci_failed:
    BLOCK_MERGE

IF unresolved_blocking_security_finding:
    BLOCK_MERGE

IF unresolved_required_review_finding:
    BLOCK_MERGE

IF required_human_approval_missing:
    BLOCK_MERGE

IF repository_protection_rule_failed:
    BLOCK_MERGE

OTHERWISE:
    ALLOW_MERGE_QUEUE_ENTRY
```

Implement enforcement through trusted CI workflows, branch protection, repository rules, and appropriate permissions—not just reviewer instructions.

For mandatory checks, a service outage must not silently count as success. Define explicit behavior for timeouts, malformed responses, missing results, permission errors, and duplicate webhook deliveries. For sensitive repositories, fail closed when mandatory verification is unavailable; emergency bypasses should be authorized and audited.

## Feedback and Rollback

### Feature flags

Feature flags can enable internal-first rollout, limit exposure, and disable problematic functionality without reverting an entire release.

They do not eliminate rollback needs. Database migrations, background jobs, and irreversible external side effects require additional recovery planning.

### Recovery checklist

Document:

- How to identify the responsible change.
- How to disable affected functionality.
- How to revert application code.
- Whether database rollback is safe.
- How to handle queued or in-flight operations.
- How to validate recovery.
- Who must be notified.

For transaction-processing systems, explicitly address duplicate processing, reconciliation, and consistency across dependent systems.

## Continuous Evaluation

### Build a historical defect dataset

Collect examples from production incidents, emergency fixes, reverted PRs, validated review findings, test-discovered regressions, and security incidents.

For each example, record the original PR and diff, the defect, failure scenario, expected finding, severity, whether the PR introduced it, and which review layer could have caught it. Have engineers familiar with the affected system validate the examples.

### Measure detection quality

**Precision**

`Precision = Valid findings / All reported findings`

**Recall**

`Recall = Known defects detected / Total known defects`

**False-positive rate**

Define the denominator precisely—for example, the proportion of clean PRs that receive at least one false-positive finding.

**Review convergence**

Measure the number of automated review-fix cycles needed before a PR satisfies policy.

**Regression escape rate**

Track defects that reach later testing or production despite passing earlier review stages.

### Prevent evaluation leakage

Distinguish bugs in the original PR from bugs introduced by a fix, unrelated historical defects, valid findings previously overlooked, and false positives.

Version evaluation datasets and evaluate changes to the evaluator itself. Whenever prompts, models, tools, or policies change, rerun the evaluation suite before broad rollout.

### Fix recurring defects at the source

Consider adding a lint rule, static-analysis rule, test, agent skill, repository guidance, CI validation, or better architectural documentation. Prefer deterministic enforcement for rules that can be expressed reliably as code.

## Repository Configuration

### `CLAUDE.md`

Use this file for concise project context.

```markdown
# Project Context

## Purpose
Describe the system and its primary responsibilities.

## Architecture
Explain major modules and boundaries.

## Development
Document build, test, and local development commands.

## Engineering Rules
- Preserve backward compatibility where required.
- Validate external inputs.
- Handle errors explicitly.
- Avoid logging sensitive information.
- Add tests for behavioral changes.

## Verification
List checks required before opening a PR.

## Additional Documentation
Link to detailed architecture and operational documentation.
```

Keep frequently needed instructions concise. Put detailed reference material in dedicated documents.

### `REVIEW.md`

```markdown
# Code Review Guidelines

## Highest Priority
- Incorrect business logic.
- Data loss or corruption.
- Security vulnerabilities.
- Concurrency and ordering defects.
- Breaking public interfaces.
- Failure-handling and recovery problems.

## Review Requirements
- Identify the exact file and line.
- Explain a concrete failure scenario.
- State whether the PR introduced the defect.
- Suggest a correction when practical.
- Avoid unsupported or purely stylistic findings.

## Special Considerations
- Review sensitive configuration changes carefully.
- Check tests against intended behavior.
- Inspect relevant call sites and dependencies.
- Identify assumptions that could fail in production.

## Output
Provide a concise summary and actionable findings ordered by severity.
If no actionable findings exist, state that explicitly.
```

These files should evolve as recurring defects are discovered. They do not replace tests, repository permissions, or enforceable policies.

### Pull request template

```markdown
## Summary
What changed and why?

## Testing
- [ ] Unit tests
- [ ] Integration tests
- [ ] Build and static checks
- [ ] Other relevant validation

Commands executed and results:

## Risk Assessment
- Risk level:
- Potential failure modes:
- Affected systems or dependencies:

## Rollback Plan
How can this change be safely disabled or reverted?

## Additional Notes
Document migration considerations, feature flags, or operational requirements.
```

The author remains responsible for the accuracy of this information, even when an agent drafts it.

## Implementation Roadmap

### Phase 1: Establish a baseline

- Measure PR volume and review turnaround.
- Track human approvals and comments.
- Measure reverts and incidents.
- Identify recurring defect categories.
- Identify high-risk repositories and components.
- Collect representative historical defects.

**Exit criteria:** baseline dashboard and initial evaluation dataset exist.

### Phase 2: Automate local verification

- Add build and test instructions to repository guidance.
- Configure formatters and linters.
- Add automated diff inspection and fresh-context review.
- Require evidence for verification claims.
- Capture failures and unresolved checks.

**Exit criteria:** the workflow consistently performs intended local checks.

### Phase 3: Introduce AI PR review

- Integrate an AI reviewer into one repository.
- Define a structured finding format.
- Require file and line references.
- Deduplicate findings and filter unsupported comments.
- Track accepted, rejected, and fixed findings.
- Add review-cycle limits.

**Exit criteria:** useful defect detection with acceptable noise on the evaluation dataset.

### Phase 4: Enforce security and CI policies

- Configure branch protection.
- Add automated security analysis.
- Require successful CI.
- Require resolution of mandatory review threads.
- Protect sensitive paths.
- Define behavior for reviewer outages.
- Audit approval and bypass behavior.

**Exit criteria:** required checks are enforced by repository controls.

### Phase 5: Introduce risk-based approvals

- Define low-, medium-, high-, and critical-risk changes.
- Identify sensitive paths and code owners.
- Implement mandatory path rules.
- Introduce risk classification.
- Define approval requirements per tier.
- Evaluate misclassification rates.
- Start with conservative policies.

**Exit criteria:** risk classifications and approval decisions are auditable.

### Phase 6: Improve deployment and recovery

- Establish a merge queue.
- Introduce feature flags where appropriate.
- Implement staged rollout procedures.
- Build a documented revert mechanism.
- Monitor production health and business correctness.
- Rehearse recovery procedures.

**Exit criteria:** representative failures can be detected and recovered from within operational objectives.

### Phase 7: Continuous optimization

- Expand the historical defect dataset.
- Evaluate changes to review behavior.
- Measure false positives and review convergence.
- Identify recurring defect patterns.
- Add deterministic checks where possible.
- Reassess approval requirements using operational evidence.

**Exit criteria:** changes demonstrate measurable benefits without unacceptable regressions.

## Metrics and Observability

Measure engineering effort, defect detection, and production outcomes together.

| Metric | What it measures |
|---|---|
| PR throughput | PRs merged over time |
| Time to merge | Time from PR creation to merge |
| Human approvals per PR | Human review effort |
| 95th-percentile approvals | Effort for unusually complex PRs |
| Human comments per PR | Amount of human review discussion |
| Time waiting for a human | Approval bottlenecks |
| AI findings per PR | Automated finding volume |
| Validated finding rate | Proportion of findings accepted as valid |
| Review-fix cycles | Automated correction rounds |
| Revert rate | Frequency of changes requiring reversion |
| Incident rate | Production incidents associated with changes |
| Detection latency | Time between failure and detection |
| Recovery time | Time to restore acceptable operation |
| Evaluation recall | Ability to detect known defects |
| Evaluation precision | Proportion of reported findings that are valid |

A reduction in review time is not a success if it materially increases incidents or reverts. Compare against a baseline with consistent definitions, and account for PR volume, severity, deployment frequency, and system complexity.

## Best Practices

### Do

- Verify generated code before opening a PR.
- Review changes against original intent.
- Provide reviewers with repository context.
- Independently validate important findings.
- Require deterministic CI checks.
- Enforce risk-based human approval.
- Limit automated review cycles.
- Preserve clear ownership.
- Make rollback predictable.
- Use real incidents to improve evaluation.
- Track human effort and production quality together.

### Avoid

- Requiring humans to inspect every line regardless of risk.
- Treating AI approval as a replacement for mandatory human approval.
- Trusting claims that tests passed without execution evidence.
- Allowing unlimited review-fix loops.
- Reporting speculative issues without actionable evidence.
- Letting AI risk scores override repository policy.
- Treating a successful build as proof of correctness.
- Optimizing review speed while ignoring incident rates.
- Using feature flags as a substitute for data-recovery planning.

## Future Enhancements

### Context-aware review

Retrieve relevant architecture documents, prior incidents, API contracts, and related code to improve review quality.

### Automated regression-test generation

Use identified defects to propose tests that reproduce failures before suggesting a fix.

### Change-impact analysis

Identify affected modules, downstream consumers, database dependencies, and operational workflows.

### Adaptive review depth

Choose review strategies based on risk, uncertainty, and measured reviewer performance.

### Incident-driven policy improvement

Propose tests, review rules, and documentation updates after incidents, with human validation before applying policy changes.

### Controlled autonomous merging

Permit automatic merging only for explicitly eligible low-risk changes when all deterministic checks and required review policies pass. Start with narrow criteria and expand only when evaluation supports it.

## Conclusion

The future of code review is not simply to have AI read more code. It is to build a system that combines automated verification, context-aware defect detection, risk-based human judgment, strong repository controls, and fast recovery.

**Reduce the human effort required per change without increasing the cost or frequency of failures.**

Start with one repository, establish a measurable baseline, automate routine verification, evaluate review quality against real defects, and expand incrementally as evidence supports it.

---

*Inspired by the article “How the Claude Code team does code review.” The architecture, prompts, configuration examples, and roadmap in this README are implementation recommendations, not a claim to reproduce Anthropic's exact internal implementation.*
