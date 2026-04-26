# Architecture — Artifact Data Flow

This document visualizes how documents flow through the framework: who produces them, who consumes them, and how state is carried forward between sessions. For the methodology behind the flow, see [WORKFLOW.md](WORKFLOW.md). For the prompt that drives it, see [BOOTSTRAP_PROMPT.md](BOOTSTRAP_PROMPT.md).

The framework's "data" is Markdown files. Every arrow below is a file being written, read, or updated.

---

## 1. High-Level View

The pipeline turns a one-line idea into a self-resuming build loop. PLANNING.md is the gating artifact — nothing downstream is generated until it's locked.

```mermaid
flowchart TD
    Idea([User project idea<br/>+ Phase A clarifying answers])

    Idea --> SkillAcq[Skill acquisition<br/>Phase B subagent]
    SkillAcq --> SkillFiles[(skill-*.md files<br/>+ Skill Manifest)]

    Idea --> PlanAgent[Planning + research<br/>Phase C subagent]
    SkillFiles -. loaded into .-> ReviewAgents
    PlanAgent --> Planning[PLANNING.md<br/>draft]

    Planning --> ReviewAgents[5 parallel review subagents<br/>Phase D]
    ReviewAgents --> Questions[/10 review questions/]
    Questions --> UserAns[/User answers/]
    UserAns --> Reconcile[Reconciliation subagent<br/>Phase E]
    Reconcile --> Planning
    Reconcile -. new questions? .-> Questions

    Planning ==locked==> SpecGen[Spec generation<br/>Phase F]

    subgraph Stage2 [Stage 2 / Phase F outputs]
        DEVPLAN[DEVPLAN.md]
        WORKPLAN[WORKPLAN.md]
        CLAUDE[CLAUDE.md]
        DEVLOG[DEVLOG.md]
        KICKOFF[KICKOFF_PROMPT.md]
    end
    SpecGen --> Stage2

    Stage2 -. optional<br/>Phase G .-> POC[poc/ folder<br/>simplified copies of<br/>PLANNING / DEVPLAN /<br/>CLAUDE / DEVLOG / KICKOFF]
    POC --> Lessons[LESSONS_LEARNED.md]
    Lessons -. refines .-> Planning
    Lessons -. reshapes .-> DEVPLAN

    KICKOFF ==pasted by user==> Cycle{{Build cycle<br/>Stage 4}}
    CLAUDE --> Cycle
    DEVLOG --> Cycle
    DEVPLAN --> Cycle
    WORKPLAN --> Cycle
    SkillFiles -. loaded per phase .-> Cycle
    Cycle ==appends entry==> DEVLOG
    Cycle -. drift detected .-> Planning

    classDef artifact fill:#eef,stroke:#558,color:#000
    classDef agent fill:#efe,stroke:#585,color:#000
    classDef user fill:#fee,stroke:#855,color:#000
    class Planning,DEVPLAN,WORKPLAN,CLAUDE,DEVLOG,KICKOFF,SkillFiles,Lessons,POC artifact
    class SkillAcq,PlanAgent,ReviewAgents,Reconcile,SpecGen,Cycle agent
    class Idea,Questions,UserAns user
```

**Reading the diagram:**

- **Solid arrows** = a file is written or a step always happens.
- **Dotted arrows** = conditional: optional stage, loop, or feedback edge.
- **Double-line arrows (`==>`)** = a gate the user explicitly triggers (locking PLANNING, pasting KICKOFF_PROMPT).
- **Feedback edges** are first-class: `LESSONS_LEARNED.md → PLANNING.md`, `Cycle → DEVLOG.md`, and the `drift detected → PLANNING.md` edge are how the framework stays consistent over time. PLANNING.md should always describe the system as it is, not as it was originally imagined.

---

## 2. Detail View — Field-Level Flow Between Artifacts

The high-level view shows *which* files connect. This view shows *what* moves between them. Each label on an edge names the section, list, or piece of state that the downstream artifact consumes.

```mermaid
flowchart LR
    subgraph Source [Inputs]
        IdeaIn[/Project idea +<br/>7 clarifying answers/]
    end

    subgraph PlanArt [PLANNING.md sections]
        Reqs[Requirements<br/>+ confidence levels]
        Arch[System Architecture<br/>+ component deep-dives]
        Ext[External Integrations<br/>API endpoints, auth, SDKs]
        Risk[Risks &amp; Open Questions]
        Scope[MVP Scope]
        Decisions[Resolved Decisions<br/>from Phase E]
    end

    subgraph DevArt [DEVPLAN.md per phase]
        What[WHAT: deliverables]
        Why[WHY: user value]
        How[HOW: approach + libs]
        Deps[DEPENDENCIES:<br/>prior phases]
        Tests[TEST CASES:<br/>20-40 named tests]
    end

    subgraph WorkArt [WORKPLAN.md per phase]
        BSkills[Builder skills]
        RSkills[Reviewer skills]
        Parallel[Parallelism / blocked-by]
    end

    subgraph CtxArt [CLAUDE.md]
        Stack[Tech stack]
        Settled[Settled decisions]
        Protocol[Dev cycle protocol]
        FileMap[Where to find things]
    end

    subgraph LogArt [DEVLOG.md]
        Status[Current cycle + phase]
        TestRes[Tests written / passing]
        Blockers[Blockers]
        Next[What to do next]
    end

    subgraph KickArt [KICKOFF_PROMPT.md]
        ReadStep[1. Read CLAUDE + DEVLOG + DEVPLAN + WORKPLAN]
        AskStep[2. Ask one-time questions]
        BuildStep[3. Builder→Test→Reviewer→Log loop]
        StopStep[4. Stop cleanly + summarize]
    end

    subgraph CycleRun [One build cycle]
        Builder[Builder subagent]
        Reviewer[Reviewer subagent]
        TestRun[Test runner]
    end

    SkillsIn[(skill-*.md files)]

    IdeaIn --> Reqs
    IdeaIn --> Ext
    Decisions -. close .-> Risk

    Reqs --> What
    Reqs --> Tests
    Arch --> How
    Arch --> Deps
    Ext --> How
    Scope --> What
    Decisions --> Settled

    DevArt --> BSkills
    DevArt --> RSkills
    Parallel -. schedules .-> CycleRun

    Stack --> Builder
    Settled --> Builder
    Protocol --> CycleRun
    FileMap -. orient .-> Builder

    Status --> ReadStep
    Next --> ReadStep
    Blockers --> AskStep

    KickArt ==paste==> CycleRun

    Tests --> Builder
    What --> Builder
    BSkills -. load .-> Builder
    RSkills -. load .-> Reviewer
    SkillsIn -. resolved by name .-> BSkills
    SkillsIn -. resolved by name .-> RSkills

    Builder --> TestRun
    TestRun -. fail x3 .-> Blockers
    TestRun -. pass .-> Reviewer
    Reviewer -. changes requested .-> Builder
    Reviewer -. approved .-> Status
    CycleRun --> TestRes

    classDef section fill:#eef,stroke:#558,color:#000
    classDef agent fill:#efe,stroke:#585,color:#000
    classDef input fill:#fee,stroke:#855,color:#000
    class Reqs,Arch,Ext,Risk,Scope,Decisions,What,Why,How,Deps,Tests,BSkills,RSkills,Parallel,Stack,Settled,Protocol,FileMap,Status,TestRes,Blockers,Next,ReadStep,AskStep,BuildStep,StopStep section
    class Builder,Reviewer,TestRun,CycleRun agent
    class IdeaIn,SkillsIn input
```

**What this shows that the high-level view doesn't:**

- **Tests are the load-bearing edge.** `PLANNING.md → Requirements → DEVPLAN.md → Tests → Builder` is the spine of the framework. The named test cases in DEVPLAN.md are the actual specification handed to the builder subagent — code is only ever written to make those named tests pass.
- **WORKPLAN.md is a name-resolver.** Its only job is to map phase IDs to the subset of `skill-*.md` files that should be loaded into the builder/reviewer subagents for that phase. Skill files themselves are downloaded once in Phase B and re-used for the lifetime of the project.
- **CLAUDE.md is read every session; DEVLOG.md is written every session.** The pair forms the framework's persistent memory. Settled decisions go to CLAUDE.md (so they're never re-litigated); per-cycle state goes to DEVLOG.md (so the next session can resume).
- **The fail-x3 → Blockers edge is the only path that escapes the cycle.** Three failed test runs on a phase means the cycle stops, the blocker is logged, and the user is asked to intervene before the next paste of KICKOFF_PROMPT.md.
- **KICKOFF_PROMPT.md never reads from itself.** It is purely a launcher — the build cycle consumes the four other Stage-2 artifacts directly.

---

## Where each artifact is templated

Every artifact in the diagrams above starts from a template in [`templates/`](templates/):

| Artifact | Template |
|---|---|
| PLANNING.md | `templates/PLANNING_TEMPLATE.md` |
| DEVPLAN.md | `templates/DEVPLAN_TEMPLATE.md` |
| WORKPLAN.md | `templates/WORKPLAN_TEMPLATE.md` |
| CLAUDE.md | `templates/CLAUDE_TEMPLATE.md` |
| DEVLOG.md | `templates/DEVLOG_TEMPLATE.md` |
| KICKOFF_PROMPT.md | `templates/KICKOFF_PROMPT_TEMPLATE.md` |
| poc/* | `templates/POC_SETUP_TEMPLATE.md` |
| LESSONS_LEARNED.md | `templates/LESSONS_LEARNED_TEMPLATE.md` |

Templates use `{{PLACEHOLDER}}` syntax; the bootstrap pipeline fills them in.
