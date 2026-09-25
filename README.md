# coworker

Absolutely — here’s a polished README-ready version that positions this as an engineering platform, not just another coding agent.

Team Engineering Agent

From requirement to review-ready Pull Request — using your team’s engineering knowledge.

Team Engineering Agent is an AI-powered engineering execution platform that turns a software requirement into review-ready code changes across one or more repositories.

It combines team-owned engineering skills, intelligent model routing, MCP, developer tooling, CI/CD, Jira, SonarQube, and Git workflows to automate the repetitive parts of the software development lifecycle while keeping human engineers in control of the final code review and merge decision.

⸻

The Vision

A developer should be able to provide a requirement such as:

Add support for real-time margin adjustment events for options expiry.

And the engineering agent should be able to:

Requirement
    ↓
Understand requirement
    ↓
Identify affected projects
    ↓
Load SME-approved project skills
    ↓
Create implementation plan
    ↓
Create Jira tasks
    ↓
Implement changes
    ↓
Generate / update tests
    ↓
Run build & tests
    ↓
Run SonarQube
    ↓
Resolve automated quality issues
    ↓
Execute CI/CD pipeline
    ↓
Create Pull Request(s)
    ↓
         👤 HUMAN REVIEW

The agent does not merge the Pull Request.

The final endpoint is always human review.

⸻

Why?

Modern coding agents are becoming increasingly capable at writing code, but enterprise engineering involves much more than generating code.

A typical engineering task may require:

* Understanding existing architecture
* Finding the correct repositories
* Following team-specific coding patterns
* Understanding internal frameworks
* Creating Jira tickets
* Modifying multiple services
* Writing tests
* Running builds
* Resolving SonarQube findings
* Running CI/CD pipelines
* Investigating failures
* Creating Pull Requests
* Maintaining traceability between requirement, Jira, code and PR

Different engineering teams also have different rules, patterns and domain knowledge.

Team Engineering Agent aims to provide an enterprise engineering execution layer that understands those differences.

⸻

Core Concepts

1. Team-Owned Engineering Skills

Every engineering team can onboard its own project skills.

A skill represents approved engineering knowledge that the agent can use while implementing a requirement.

Examples:

@rtm.kafka
@rtm.margin-calculation
@rtm.database
@rtm.testing
@rtm.mainframe-migration

A skill can contain:

* Engineering standards
* Architecture patterns
* Coding patterns
* Approved libraries
* Examples
* Anti-patterns
* Testing requirements
* Validation commands
* Deployment information
* Runbooks
* Domain knowledge
* Repository-specific rules

Example:

skill:
  name: rtm-kafka-consumer
  version: 2.1
  owner: RTM Platform
  status: SME_APPROVED
scope:
  repositories:
    - rtm-trade-consumer
    - rtm-margin-engine
rules:
  - Use the approved Kafka abstraction
  - Use manual offset commits
  - Use tradeReference as the partition key
  - Configure DLQ for failures
testing:
  required:
    - unit
    - integration
    - kafka-contract
validation:
  commands:
    - ./gradlew test
    - ./gradlew integrationTest
    - ./gradlew sonar

Skills are versioned and can follow a governance lifecycle:

DRAFT
  ↓
REVIEW
  ↓
SME APPROVED
  ↓
ACTIVE
  ↓
DEPRECATED

This allows teams to control what engineering practices the AI is allowed to use.

⸻

2. Intelligent Model Router

The platform does not use the most expensive model for every task.

Instead, a Model Router selects the appropriate model based on task complexity.

                  MODEL ROUTER
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
     Small           Medium        Reasoning
     Model            Model          Model

Typical routing

Task	Model Tier
Requirement classification	Small
Jira description	Small
Skill retrieval	Small
Repository discovery	Small
Code search	Small
Simple code change	Medium
Test generation	Medium
PR description	Small
Architecture analysis	Reasoning
Cross-repository decomposition	Reasoning
Complex bug investigation	Reasoning
Complex Sonar remediation	Reasoning
Complex pipeline failure	Reasoning

The goal is:

Use the cheapest model that can reliably complete the task.

⸻

3. Predictable AI Consumption

A key design goal is predictable AI usage for common engineering tasks.

For the normal execution path:

Requirement
    ↓
Standard task graph
    ↓
Predictable model routing
    ↓
Predictable AI consumption

AI usage may increase when:

* A complex problem requires deeper reasoning
* Tests repeatedly fail
* Sonar remediation requires architectural changes
* CI/CD pipelines fail
* The agent needs to retry an implementation
* A user requests implementation changes
* The user provides additional iterations

Conceptually:

Baseline AI Cost
        +
Failure Recovery Cost
        +
User Iteration Cost
        =
Total AI Cost

This creates a predictable baseline while allowing the platform to spend additional reasoning capacity when the problem actually requires it.

⸻

4. Adaptive Model Escalation

The agent starts with the appropriate lower-cost model and escalates when necessary.

             Small Model
                  │
                  ▼
               Attempt
                  │
           ┌──────┴──────┐
           │             │
          PASS          FAIL
           │             │
           ▼             ▼
        Continue      Medium Model
                         │
                      Attempt
                         │
                   ┌─────┴─────┐
                   │           │
                  PASS        FAIL
                   │           │
                   ▼           ▼
                Continue   Reasoning Model

This prevents expensive reasoning models from becoming the default for simple engineering operations.

⸻

5. Multi-Repository Engineering

A single requirement can affect multiple projects.

For example:

Requirement
     │
     ▼
Dependency Analysis
     │
     ├── Trade Service
     │       └── Repository A
     │
     ├── Position Service
     │       └── Repository B
     │
     ├── Margin Engine
     │       └── Repository C
     │
     └── Reporting Service
             └── Repository D

The agent can create separate implementation tasks and Pull Requests:

                  Master Requirement
                         │
              ┌──────────┼──────────┐
              ▼          ▼          ▼
           Task A      Task B      Task C
           Repo A      Repo B      Repo C
              │          │          │
             PR A       PR B       PR C

The relationship between the requirement, Jira tasks and Pull Requests is maintained throughout the execution.

⸻

6. Jira Integration

The agent can convert a requirement into an actionable Jira hierarchy.

Example:

RTM-8420
Real-time margin adjustment support
├── RTM-8421
│   Trade event changes
│
├── RTM-8422
│   Position service changes
│
└── RTM-8423
    Margin engine changes

Each task contains:

* Requirement context
* Implementation details
* Acceptance criteria
* Relevant engineering skills
* Affected repository
* Testing requirements
* Dependencies

⸻

7. Engineering Execution

For each task, the agent can:

Find Repository
      ↓
Create Branch
      ↓
Understand Existing Code
      ↓
Load Relevant Skills
      ↓
Create Implementation Plan
      ↓
Modify Code
      ↓
Generate / Update Tests
      ↓
Run Tests
      ↓
Run Build
      ↓
Run SonarQube
      ↓
Resolve Findings
      ↓
Run CI/CD Pipeline
      ↓
Create Pull Request

The agent should operate within explicit permissions and repository policies.

⸻

8. Automated SonarQube Remediation

SonarQube findings can be automatically classified and resolved where appropriate.

For example:

Sonar Finding
      ↓
Classify
      │
      ├── Simple/local fix
      │        ↓
      │     Small Model
      │
      └── Architectural/complex fix
               ↓
          Reasoning Model

After every fix:

Fix
 ↓
Test
 ↓
Sonar
 ↓
Pass → Continue
Fail → Diagnose / Escalate

The agent should operate with configurable retry and escalation limits to prevent uncontrolled execution.

⸻

9. Human-in-the-Loop

The platform is designed around a clear boundary:

AI executes. Humans review and decide.

The agent can:

* Create Jira
* Modify code
* Run tests
* Fix quality issues
* Run pipelines
* Create PRs

The agent does not automatically merge the Pull Request.

Final ownership remains with the engineer.

AI
 │
 ├── Analyze
 ├── Plan
 ├── Implement
 ├── Validate
 └── Create PR
          │
          ▼
       HUMAN
          │
       Review
          │
    ┌─────┴─────┐
    ▼           ▼
  Approve     Request
   / Merge    Changes

⸻

MCP Architecture

The platform is designed to work as an MCP-based engineering capability.

Coding tools such as IDE agents, Claude Code, Codex and other MCP-compatible clients can interact with the engineering platform without needing to understand the underlying enterprise tooling.

                  Coding Tools
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      IDE          Claude Code       Codex
        │              │              │
        └──────────────┼──────────────┘
                       │
                      MCP
                       │
              ┌────────▼────────┐
              │ Team Engineering│
              │     Agent       │
              └────────┬────────┘
                       │
              ┌────────▼────────┐
              │  Orchestrator   │
              └────────┬────────┘
                       │
              ┌────────▼────────┐
              │  Model Router   │
              └────────┬────────┘
                       │
              ┌────────▼────────┐
              │   Tool Gateway  │
              └────────┬────────┘
                       │
       ┌───────────────┼────────────────┐
       ▼               ▼                ▼
      Git             Jira             CI/CD
       │                                │
       ▼                                ▼
    SonarQube                       Pipelines

⸻

MCP Capabilities

The platform can expose granular capabilities such as:

Requirement

analyze_requirement()
decompose_requirement()
identify_projects()
identify_skills()

Skills

search_project_skills()
get_skill()
get_architecture()
get_coding_standard()

Jira

create_epic()
create_story()
create_task()
update_jira()
link_jira()

Repository

find_repository()
search_code()
read_repository()
create_branch()
apply_patch()
commit_changes()
create_pr()

Validation

run_tests()
run_build()
run_sonar()
get_sonar_findings()
run_pipeline()
get_pipeline_status()

Engineering Execution

implement_task()
validate_task()
create_pull_request()

The MCP layer allows external coding agents to consume these capabilities while the Team Engineering Agent controls orchestration, model routing, skills and enterprise policies.

⸻

CLI Experience

The primary developer interface can be a simple CLI.

engagent "Add support for real-time margin adjustment events"

The agent analyzes the requirement:

Analyzing requirement...
Affected projects:
  ✓ trade-service
  ✓ position-service
  ✓ margin-engine
Applicable skills:
  ✓ @rtm.kafka@2.1
  ✓ @rtm.margin@3.4
  ✓ @rtm.testing@4.2
Proposed implementation:
  3 Jira tasks
  3 repositories
  3 Pull Requests
Continue? [Y/n]

After execution:

Implementation complete.
Jira:
  RTM-8420
Pull Requests:
  trade-service       PR #1832
  position-service    PR #1833
  margin-engine       PR #1834
Validation:
  Tests       ✓
  Build       ✓
  Sonar       ✓
  Pipeline    ✓
Nothing has been merged.
Human review required.

⸻

Agent State Machine

The agent follows an explicit execution lifecycle:

REQUIREMENT_RECEIVED
        ↓
ANALYZING
        ↓
SKILLS_RESOLVED
        ↓
PLAN_CREATED
        ↓
USER_APPROVAL
        ↓
JIRA_CREATED
        ↓
IMPLEMENTING
        ↓
TESTING
        ↓
SONAR_VALIDATION
        ↓
PIPELINE
        ↓
PR_CREATED
        ↓
HUMAN_REVIEW

Failures can trigger controlled remediation:

TEST_FAILURE
      ↓
DIAGNOSE
      ↓
FIX
      ↓
RETEST

with configurable:

max_attempts
max_token_budget
max_model_tier
execution_timeout

⸻

Engineering Context

Each execution maintains a structured task state:

Task
 ├── Requirement
 ├── Jira
 ├── Skills + versions
 ├── Implementation plan
 ├── Model decisions
 ├── Context retrieved
 ├── Files changed
 ├── Tests
 ├── Sonar findings
 ├── Pipeline executions
 ├── User feedback
 └── Pull Requests

This allows subsequent user iterations to reuse existing context rather than restarting the entire reasoning process.

For example:

Initial implementation
        ↓
User: "Use the existing margin pipeline instead"
        ↓
Delta analysis
        ↓
Re-plan affected components
        ↓
Modify only required changes

⸻

Security and Governance

Enterprise engineering requires explicit boundaries.

The agent should operate through configurable permissions.

Example:

READ_REPOSITORY       ✓
CREATE_BRANCH         ✓
WRITE_CODE            ✓
CREATE_JIRA           ✓
RUN_TESTS             ✓
RUN_PIPELINE          ✓
CREATE_PR             ✓
MERGE_PR              ✗
PRODUCTION_DEPLOY     ✗

All significant actions should be auditable.

Example execution record:

{
  "requirement": "Add XYZ",
  "agent": "rtm-engineering-agent",
  "skills": [
    "rtm.kafka@2.1",
    "rtm.margin@3.4"
  ],
  "repositories": [
    "margin-engine",
    "trade-service"
  ],
  "jira": [
    "RTM-8421"
  ],
  "pull_requests": [
    "PR-1832",
    "PR-921"
  ],
  "tests": "PASSED",
  "sonar": "PASSED",
  "pipeline": "PASSED",
  "merged": false
}

⸻

Model Routing Architecture

The Model Router should treat model selection as an engineering optimization problem.

Task
 ↓
Classify Complexity
 ↓
Estimate Context
 ↓
Evaluate Skill Confidence
 ↓
Check Previous Attempts
 ↓
Select Model
 ↓
Assign Budget
 ↓
Execute
 ↓
Evaluate Result
 ↓
Escalate if Required

Routing can consider:

* Task type
* Complexity
* Repository count
* Files affected
* Context size
* Skill confidence
* Previous failures
* Number of iterations
* Tool failures
* Test failures
* Sonar findings
* Model reliability
* Latency
* AI cost

The goal is not simply to minimize model cost.

The goal is:

Minimize cost per successfully completed engineering task.

⸻

Observability

The platform should track both engineering and AI metrics.

Engineering metrics

Requirements processed
Jira tasks created
Repositories modified
PRs created
PRs successfully validated
Tests passed
Pipeline success rate
Sonar remediation rate
Human iterations

AI metrics

AI credits / requirement
AI credits / PR
Tokens / task
Model distribution
Model escalation rate
Retry rate
Reasoning-model usage
Cost by task type
Cost by team

Example:

Engineering Agent Metrics
PRs created                    1,240
Successfully validated         1,103
Average AI credits / PR           42k
P50 AI credits / PR               31k
P90 AI credits / PR               69k
P99 AI credits / PR              180k
Average user iterations / PR       0.7
Average automated recoveries       1.3

This makes AI consumption measurable and predictable.

⸻

Platform Architecture

The platform can be organized into five major layers:

┌──────────────────────────────────────────────┐
│                  EXPERIENCE                  │
│ CLI / IDE / MCP / Chat / Coding Agents      │
├──────────────────────────────────────────────┤
│                 ORCHESTRATOR                 │
│ Requirement → Plan → Tasks → Execution       │
├──────────────────────────────────────────────┤
│                  SKILL ENGINE                │
│ Team Skills / SME Approval / Versioning      │
├──────────────────────────────────────────────┤
│                MODEL ROUTER                  │
│ Small / Medium / Reasoning Models            │
├──────────────────────────────────────────────┤
│                  TOOL GATEWAY                 │
│ Git / Jira / CI / Sonar / Docs / Artifacts   │
├──────────────────────────────────────────────┤
│               ENTERPRISE SYSTEMS             │
│ GitHub / GitLab / Jira / Jenkins / Sonar etc │
└──────────────────────────────────────────────┘

⸻

Long-Term Vision

The Team Engineering Agent can evolve from a coding agent into a broader Engineering Agent Platform.

Team Engineering Agent
│
├── Requirement Agent
├── Coding Agent
├── Test Agent
├── Regression Agent
├── Migration Agent
├── Incident Agent
└── Release Agent

All capabilities share:

Skills
Knowledge
Model Router
MCP
Tooling
Permissions
Audit
Observability

This creates a common engineering intelligence layer for teams while allowing every team to retain ownership of its domain-specific engineering knowledge.

⸻

Guiding Principles

1. Human ownership

AI prepares changes. Engineers own the final decision.

2. Team knowledge over generic knowledge

The agent should follow approved team practices rather than inventing new ones.

3. Cheapest capable model

Use expensive reasoning only when the task warrants it.

4. Fail forward

Use controlled retries, diagnosis and model escalation instead of blindly repeating the same operation.

5. Multi-repository by design

A requirement may naturally span multiple services and repositories.

6. Everything is auditable

Requirements, skills, model decisions, code changes, tests, pipelines and PRs should be traceable.

7. MCP-first interoperability

The engineering platform should be consumable by multiple coding agents and developer experiences.

8. Predictable AI consumption

Normal engineering workflows should have predictable AI usage, with additional consumption driven by complexity, failures and user iterations.

⸻

The End Goal

The experience should eventually be as simple as:

engagent "Implement <requirement>"

Behind that single command:

Requirement
    ↓
Team Knowledge
    ↓
Model Router
    ↓
Implementation Plan
    ↓
Jira
    ↓
Code
    ↓
Tests
    ↓
Sonar
    ↓
CI/CD
    ↓
Pull Requests
    ↓
👤 Engineer Review

The agent handles the execution.
The team defines the skills.
The model router controls the intelligence budget.
The engineer owns the final decision.

This is structured so you can paste it directly into README.md; the sections can also become the basis for the eventual architecture/design docs.
