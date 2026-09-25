# coworker

Yes. I’d frame this as a Team Engineering Agent rather than just a coding agent.

The interesting part isn’t “AI writes code.” The product is:

Give the agent a business/engineering requirement → it discovers the team’s approved engineering knowledge → decomposes the work → creates Jira → changes the right repositories → validates the changes → fixes automated findings → creates PRs → stops at human review.

And the fact that each team can onboard its own project skills makes this much more interesting than another generic coding agent.

1. The core idea

Think of it as a virtual engineering coworker for a team.

                    ┌─────────────────────────┐
                    │       USER / DEV        │
                    │                         │
                    │ "Add support for X"     │
                    └────────────┬────────────┘
                                 │
                                 ▼
                    ┌─────────────────────────┐
                    │   TEAM ENGINEERING      │
                    │        AGENT             │
                    └────────────┬────────────┘
                                 │
                ┌────────────────┼─────────────────┐
                ▼                ▼                 ▼
          Requirement       Skill Registry     Project Graph
           Analyzer         SME-approved       repos/deps/
                              knowledge          services
                │                │                 │
                └────────────────┼─────────────────┘
                                 ▼
                       ┌─────────────────┐
                       │ Task Decomposer │
                       └────────┬────────┘
                                │
              ┌─────────────────┼──────────────────┐
              ▼                 ▼                  ▼
          Jira Task A       Jira Task B        Jira Task C
          Repo A            Repo B             Repo C
              │                 │                  │
              ▼                 ▼                  ▼
          Implement         Implement          Implement
              │                 │                  │
              └─────────────────┼──────────────────┘
                                ▼
                       Build / Test / Sonar
                                │
                         ┌──────┴──────┐
                         │             │
                       FAIL          PASS
                         │             │
                         ▼             ▼
                    Agent fixes    Create PR
                         │             │
                         └──────►──────┘
                                      │
                                      ▼
                              👤 HUMAN REVIEW

The PR is the terminal point.

The agent does not merge.

⸻

2. The killer concept: Project Skills

This is what I’d make the differentiator.

Instead of giving the LLM a giant generic system prompt, every engineering team owns a Skill Pack.

For example:

rtm/
├── architecture/
├── coding-standards/
├── kafka/
├── margin-calculation/
├── database/
├── testing/
├── deployment/
├── observability/
├── migration/
└── runbooks/

Each skill describes things the agent is allowed and expected to know.

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
  - "Use company Kafka wrapper"
  - "Do not instantiate KafkaConsumer directly"
  - "Use manual offset commit"
  - "Partition key must be tradeReference"
  - "All consumers require DLQ configuration"
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
references:
  - architecture/kafka.md
  - patterns/consumer.md

Now the agent isn’t simply asking:

“What code would a good Java developer write?”

It’s asking:

“What code would this team approve?”

That’s a huge distinction.

⸻

3. Skill governance

I’d make SME approval a first-class concept.

Every skill has:

DRAFT
   ↓
REVIEW
   ↓
SME APPROVED
   ↓
ACTIVE
   ↓
DEPRECATED

And ideally:

Skill
 ├── owner
 ├── SME approvers
 ├── version
 ├── applicable repos
 ├── applicable services
 ├── rules
 ├── examples
 ├── forbidden patterns
 ├── validation commands
 └── change history

This gives you an interesting governance model:

AI autonomy is constrained by institutional knowledge.

That’s much more compelling for a bank than “let an LLM modify our repositories.”

⸻

4. The CLI

I’d make the CLI the primary developer experience.

Something like:

$ team-agent implement "Add support for XYZ margin adjustment"

The agent responds:

Analyzing requirement...
Detected:
  • RTM Margin Engine
  • Trade Service
  • Account Service
Applicable skills:
  ✓ margin-calculation@3.2
  ✓ kafka-events@2.1
  ✓ postgres-patterns@1.8
  ✓ testing-standard@4.0
Proposed implementation:
  RTM-8421
  ├── margin-engine
  │   ├── Add MarginAdjustmentCalculator
  │   └── Add unit/integration tests
  │
  ├── trade-service
  │   └── Add MarginAdjustment event
  │
  └── account-service
      └── Consume MarginAdjustment event
3 Jira tasks
3 repositories
3 pull requests
Proceed?

The user says:

yes

Then:

Creating Jira tickets...
✓ RTM-8421
✓ RTM-8422
✓ RTM-8423
Implementation started...
[1/3] trade-service
  ✓ code generated
  ✓ tests added
  ✓ tests passed
  ✓ sonar passed
  ✓ PR created
[2/3] account-service
  ✓ code generated
  ✓ tests added
  ⚠ sonar: 3 issues
  Attempting automated remediation...
  ✓ 3 issues resolved
  ✓ tests passed
  ✓ sonar passed
  ✓ PR created
[3/3] margin-engine
  ✓ code generated
  ✓ tests added
  ✓ pipeline passed
  ✓ PR created

Final:

Implementation complete.
Jira:
  RTM-8421
  RTM-8422
  RTM-8423
Pull Requests:
  trade-service    → PR #1832
  account-service  → PR #921
  margin-engine    → PR #4721
Validation:
  Tests       ✓
  Build       ✓
  Sonar       ✓
  Pipeline    ✓
Human review required.
Nothing has been merged.

That’s a very clean endpoint.

⸻

5. But the CLI shouldn’t actually do everything itself

This is where MCP becomes powerful.

I’d separate the system into:

Agent

The reasoning/orchestration layer.

MCP Server

The standardized engineering interface.

Enterprise tools

Jira, Git, CI/CD, Sonar, artifact repositories, documentation, etc.

Architecture:

                    Claude Code
                        │
                    Codex / IDE
                        │
                    VS Code
                        │
                 team-agent CLI
                        │
                        ▼
                ┌───────────────┐
                │ Team Agent    │
                │ MCP Gateway   │
                └───────┬───────┘
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
          Jira MCP    Git MCP    CI MCP
             │          │          │
             ▼          ▼          ▼
           Jira       GitHub/     Jenkins/
                      GitLab      GitHub Actions
                                  etc.
                        │
             ┌──────────┼──────────┐
             ▼          ▼          ▼
          Sonar MCP  Artifact    Knowledge
                     MCP         MCP

This means your agent isn’t locked to your CLI.

Any coding agent capable of MCP could potentially use your engineering platform.

⸻

6. MCP tools I’d expose

This is where I’d get quite opinionated.

Don’t expose one giant:

execute_everything()

Expose granular capabilities.

Requirement

analyze_requirement()
decompose_requirement()
identify_projects()
identify_skills()

Knowledge

search_project_skills()
get_skill()
validate_skill()
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
create_branch()
read_repository()
search_code()
apply_patch()
commit_changes()
create_pr()

Build

run_tests()
run_build()
run_integration_tests()
run_pipeline()
get_pipeline_status()

Quality

run_sonar()
get_sonar_findings()
apply_sonar_fix()

Review

get_pr()
get_pr_diff()
get_review_comments()

The agent can compose these.

⸻

7. The really interesting MCP primitive: implement_task

You could also expose a higher-level tool:

implement_task(
    jira_id,
    repository,
    skill_set
)

Internally:

implement_task
      │
      ├── load Jira
      ├── load skills
      ├── inspect repo
      ├── understand architecture
      ├── plan
      ├── implement
      ├── test
      ├── build
      ├── sonar
      ├── remediate
      ├── pipeline
      └── PR

So external agents can operate at different abstraction levels.

For example, Claude Code could simply say:

Use the RTM engineering agent to implement RTM-8421.

Your MCP server takes over.

⸻

8. Multi-repository orchestration

This is probably one of the strongest parts of the idea.

Requirement:

“Support real-time margin adjustment for options expiry.”

Agent determines:

Requirement
    │
    ▼
Dependency Graph
    │
    ├── Trade Service
    │       └── Repo A
    │
    ├── Position Service
    │       └── Repo B
    │
    ├── Margin Engine
    │       └── Repo C
    │
    └── Reporting
            └── Repo D

Then:

                    MASTER TASK
                        │
              ┌─────────┼─────────┐
              ▼         ▼         ▼
           TASK A     TASK B     TASK C
           Repo A     Repo B     Repo C
              │         │         │
             PR A      PR B      PR C

Each PR can have:

Jira
Requirement
Affected component
Skill versions
Implementation summary
Tests
Pipeline
Sonar
Dependencies
Risk

And the agent maintains the relationship:

RTM-8420
 ├── RTM-8421 → PR #123
 ├── RTM-8422 → PR #456
 └── RTM-8423 → PR #789

This gives engineering leadership a requirement → code lineage.

⸻

9. Don’t let the agent blindly edit code

I’d introduce an explicit state machine.

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

And failures:

TEST_FAILURE
     ↓
DIAGNOSE
     ↓
FIX
     ↓
RETEST

But importantly:

MAX_RETRY = N

You don’t want an autonomous agent burning CI for hours.

⸻

10. Add a “confidence / escalation” mechanism

This could be extremely useful in an enterprise.

The agent should be able to say:

I can implement this automatically.
Confidence: HIGH
Reason:
  ✓ Known architecture
  ✓ Existing implementation pattern found
  ✓ SME-approved skill available
  ✓ Test pattern available

Or:

Human decision required.
Reason:
  ⚠ Requirement affects margin calculation logic
  ⚠ No approved skill exists for this calculation
  ⚠ Existing implementations disagree
I created the Jira analysis but did not modify code.

This is much safer than forcing the agent to always produce code.

⸻

11. Skills should contain “examples”, not just documentation

This is important.

A skill could have:

Kafka Consumer Skill
Rules
Patterns
Anti-patterns
Examples
Tests
Validation

For example:

patterns:
  preferred:
    file: examples/TradeConsumer.java
  producer:
    file: examples/TradeProducer.java
  retry:
    file: examples/RetryableConsumer.java
anti_patterns:
    - examples/BadConsumer.java

The agent can retrieve these during implementation.

This effectively turns your existing engineering codebase into a team-specific coding model without having to train a model.

⸻

12. The “Team Skill Registry”

I would make this a separate platform.

                    Skill Registry
                         │
       ┌─────────────────┼─────────────────┐
       ▼                 ▼                 ▼
      RTM              Equities           FX
       │                 │                 │
   Java Skill        Java Skill        Java Skill
   Kafka Skill       Kafka Skill       MQ Skill
   Margin Skill      Pricing Skill     FX Skill
   DB Skill          DB Skill          DB Skill

Each team owns their namespace.

@rtm.kafka
@rtm.margin
@rtm.database
@equities.pricing
@equities.kafka
@fx.rates
@fx.messaging

Then a requirement automatically resolves skills.

⸻

13. Skill inheritance

You can even have:

Enterprise
   │
   ├── Java Standards
   ├── Security Standards
   ├── Logging Standards
   └── Testing Standards
          │
          ▼
        RTM
          │
          ├── Kafka
          ├── Margin
          ├── Trade
          └── Mainframe Migration

So teams don’t need to reinvent enterprise skills.

⸻

14. Your existing Smart Regression Agent fits beautifully

You already have the concept of a Smart Regression Agent.

I’d actually make that another capability of the same engineering-agent platform.

For example:

Team Engineering Agent
│
├── Requirement Agent
│
├── Coding Agent
│
├── Test Agent
│
├── Regression Agent
│
├── Migration Agent
│
├── Incident Agent
│
└── Release Agent

The common infrastructure is:

Skills
Memory
MCP
Tooling
Permissions
Observability
Audit

So you’re not building six independent agents.

You’re building an Engineering Agent Platform.

⸻

15. A particularly powerful concept: Engineering Knowledge Graph

Over time, the agent could construct:

Requirement
    ↓
Jira
    ↓
Service
    ↓
Repository
    ↓
Code
    ↓
Kafka Topic
    ↓
Database
    ↓
Pipeline
    ↓
Deployment
    ↓
Runbook
    ↓
Owner

Example:

Margin Calculation
      │
      ├── margin-engine
      │       ├── Git repo
      │       ├── Kafka topics
      │       ├── DB tables
      │       └── deployment
      │
      ├── trade-service
      │
      └── account-service

Now requirement analysis becomes much smarter.

The agent isn’t merely doing semantic search.

It knows:

“Changing this Java class will affect this Kafka topic, which is consumed by these three services.”

That’s where the platform starts becoming genuinely valuable.

⸻

16. Security model

For a financial institution, I’d make this a core architectural feature rather than an afterthought.

Every action gets a permission.

READ_REPO             ✓
CREATE_BRANCH         ✓
WRITE_CODE            ✓
CREATE_JIRA           ✓
RUN_PIPELINE          ✓
CREATE_PR             ✓
MERGE_PR              ✗
PROD_DEPLOY           ✗

And potentially:

Skill says:
    "Agent may modify repo"
Policy says:
    "Agent may modify repo only on feature branches"
User says:
    "Implement requirement"
Result:
    Agent can create branch + PR
    Agent cannot merge

That makes the human PR review boundary enforceable, not just a prompt instruction.

⸻

17. Audit trail

Every action should produce an event:

{
  "requirement": "Add XYZ",
  "agent": "rtm-engineering-agent",
  "skill_versions": [
    "rtm.kafka@2.1",
    "rtm.margin@3.4"
  ],
  "repositories": [
    "margin-engine",
    "trade-service"
  ],
  "jira": ["RTM-8421"],
  "prs": ["PR-1832", "PR-921"],
  "tests": "PASSED",
  "sonar": "PASSED",
  "pipeline": "PASSED",
  "merged": false
}

This becomes very useful for enterprise governance.

⸻

18. The CLI could eventually feel like this

$ engagent run
╭────────────────────────────────────────────╮
│       TEAM ENGINEERING AGENT               │
│       RTM Engineering                      │
╰────────────────────────────────────────────╯
Requirement:
> Add support for real-time margin adjustment
> events for options expiry.

Then:

🔎 Analyzing requirement
🧠 Loading approved engineering skills
🗺 Building dependency graph
📋 Creating implementation plan
Affected systems:
  trade-service
  margin-engine
  position-service
Skills:
  ✓ @rtm.kafka@2.1
  ✓ @rtm.margin@3.4
  ✓ @rtm.testing@4.2
  ✓ @enterprise.java@7.1
Plan:
  3 Jira tasks
  3 repositories
  3 PRs
Continue? [Y/n]

Then:

▶ RTM-8421 / trade-service
  ✓ branch created
  ✓ implementation
  ✓ unit tests
  ✓ integration tests
  ✓ Sonar
  ✓ pipeline
  PR #1832 created

And finally:

╭────────────────────────────────────────────╮
│ IMPLEMENTATION COMPLETE                    │
╰────────────────────────────────────────────╯
Jira
  RTM-8420
Pull Requests
  trade-service       PR #1832   ✓
  position-service    PR #1833   ✓
  margin-engine       PR #1834   ✓
Quality
  Tests               ✓
  Sonar               ✓
  Build               ✓
  Pipeline            ✓
Agent actions         47
Automated fixes        8
Human actions required 1
→ Review PRs

⸻

19. The killer developer experience

The real goal should eventually be:

Developer

$ engagent "Add support for X"

Agent

I found 3 affected projects.
I'll create:
  3 Jira tasks
  3 implementation branches
  3 PRs
Proceed?

Developer

yes

Agent

Done.
Review:
  PR #123
  PR #456
  PR #789

That’s it.

The developer doesn’t have to:

read Jira template
→ create Jira
→ find repo
→ understand architecture
→ create branch
→ code
→ write tests
→ run Gradle
→ fix Sonar
→ run pipeline
→ monitor pipeline
→ fix failure
→ push
→ create PR
→ write PR description

The agent handles the mechanical SDLC work while the engineer retains the design/review/merge decision.

⸻

20. I’d define the product around 5 layers

┌─────────────────────────────────────────────┐
│                EXPERIENCE                   │
│ CLI / IDE / Chat / Claude Code / Codex      │
├─────────────────────────────────────────────┤
│                ORCHESTRATOR                 │
│ Requirement → Plan → Tasks → Execution      │
├─────────────────────────────────────────────┤
│                SKILL ENGINE                 │
│ Team skills / SME approval / versions       │
├─────────────────────────────────────────────┤
│                MCP TOOL LAYER               │
│ Git / Jira / CI / Sonar / Docs / DB         │
├─────────────────────────────────────────────┤
│                ENTERPRISE                   │
│ GitHub/GitLab | Jira | Jenkins | Sonar      │
└─────────────────────────────────────────────┘

The MCP layer is the interoperability layer.

The Skill Engine is the institutional knowledge layer.

The Orchestrator is the intelligence layer.

The CLI/IDE is simply the interface.

⸻

21. MVP I’d actually build

Don’t start with the entire platform.

Build:

Phase 1 — Single repo

Requirement
    ↓
Skill retrieval
    ↓
Code implementation
    ↓
Tests
    ↓
Sonar
    ↓
PR

CLI:

engagent "Add XYZ"

⸻

Phase 2 — Jira

Requirement
    ↓
Jira
    ↓
Implementation
    ↓
PR

⸻

Phase 3 — MCP

Expose:

search_skills
get_project_context
implement_task
run_tests
run_pipeline
create_pr

Now Claude Code/Codex/other coding clients can consume your platform.

⸻

Phase 4 — Multi-repo

Requirement
       ↓
Dependency analysis
       ↓
       ├── Repo A → PR A
       ├── Repo B → PR B
       └── Repo C → PR C

⸻

Phase 5 — Team Skill Marketplace

Enterprise Skills
      ↓
Team Skills
      ↓
Project Skills

with SME approval/versioning.

⸻

22. The bigger vision

I wouldn’t position this internally as:

“An AI coding agent.”

That’s crowded and relatively easy to replicate.

I’d position it as:

“An enterprise engineering execution layer that converts approved requirements into reviewable software changes.”

Or more simply:

Requirement → Jira → Code → Validation → PR

with team-specific engineering knowledge controlling how that work gets done.

And MCP makes the really compelling extension:

               Engineering Agent
                       ▲
                       │ MCP
          ┌────────────┼────────────┐
          │            │            │
       Codex      Claude Code     IDE
          │            │            │
          └────────────┼────────────┘
                       │
                  Team Skills
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
        Jira          Git          CI/CD

That means you don’t have to compete with coding agents. You can make your platform the enterprise engineering control plane that coding agents plug into.

For an RTM environment specifically, the first killer demo I’d build would be:

“Implement a new RTM event-driven requirement across 2–3 services using SME-approved Kafka, margin-calculation and testing skills, create the Jira hierarchy, generate separate PRs, run the full validation pipeline, automatically remediate Sonar/test issues within policy, and hand the engineer three review-ready PRs.”

That is a much more compelling internal GenAI story than a generic “AI writes Java code.”

Would you like to shape this next as an MVP architecture or as a concrete CLI/MCP command design?
