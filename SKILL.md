---
name: repo-onboarding-questions
description: "Generate the questions an analyst needs to ask a human when inheriting a data science project — specifically the questions that CANNOT be answered from the codebase, data, and artifacts alone. Use this whenever someone is taking over, inheriting, onboarding onto, or auditing an unfamiliar data science / analytics codebase and needs to figure out what to ask the previous owner, original author, or stakeholders. Trigger on phrases like \"I just inherited this project\", \"taking over this pipeline\", \"onboarding onto this repo\", \"what should I ask about this codebase\", or \"knowledge transfer.\" This skill SURFACES questions; it does not answer them — the answers live in a human's head, a ticket, or a dead Slack channel."
---
 
# Question Generation
 
When an analyst inherits a data science project, the dangerous unknowns are not the things the code does wrong — those are findable by reading. The dangerous unknowns are the **decisions, constraints, and context that left no trace in the artifacts**: why a cutoff was chosen, which data source is authoritative, what the client actually agreed to, what got tried and abandoned. This skill reads a codebase and its artifacts and emits the questions the analyst must take to a human.
 
## What this skill does and does not do
 
- **Does**: surface a prioritized list of questions that cannot be answered from code, data, configs, commit history, or documentation alone.
- **Does NOT**: answer those questions, guess the answers, or fabricate the missing context. If the answer is genuinely recoverable from the artifacts, it is not a question for a human — note where it lives instead.
The single most important discipline: **do not hallucinate provenance.** If you cannot tell from the artifacts why something is the way it is, that absence IS the finding. Flag it as a question; do not invent a plausible rationale.
 
## The core test for every candidate question
 
Before emitting a question, apply this filter:
 
> Could a careful analyst answer this by reading the code, data, configs, git log, and docs in front of them?
 
- **Yes** → not a question for a human. Either answer it inline (with the file/line as evidence) or drop it.
- **No** → keep it. It belongs in the output.
- **Partially** ("the code shows *what* but not *why*") → keep it, and phrase it to target the missing half.
The most common failure mode is emitting questions whose answers are sitting in the repo. The second most common is emitting "why" questions where the why is actually documented. Read before you ask.
 
## Question categories to probe
 
Work through these systematically. Each targets a class of context that typically evaporates when a project changes hands.
 
### 1. Provenance & authority
- Which data source is authoritative when two disagree? (e.g., a staging table and a delivered extract both exist — which is canonical?)
- Where did the raw inputs originate, and is that source still live / refreshing?
- Are there manual data-correction steps that happened outside the code?
### 2. Decisions without recorded rationale
- Why this date cutoff / filter threshold / exclusion rule? (Code shows the value; rarely the reason.)
- Why this modelling choice / weighting scheme / imputation method over the obvious alternative?
- Hard-coded constants and magic numbers — who set them, on what basis, and are they still valid?
### 3. Business / stakeholder context
- What did the client or stakeholder actually agree to? (Scope, definitions, deliverable format.)
- What does "done" mean for this project? What's the acceptance criterion?
- Are there definitions that look standard but are bespoke here (e.g., a custom definition of "active user" or "complete record")?
### 4. The graveyard (abandoned paths)
- What was tried and deliberately abandoned? (Saves the inheritor from re-walking dead ends.)
- Are there commented-out blocks or orphaned scripts that represent a rejected approach vs. a TODO?
- Known issues the previous owner was living with but never wrote down?
### 5. Environment & external dependencies
- What runs this, on what schedule, triggered by whom or what?
- Undocumented environment assumptions: credentials, network paths, a specific machine, a manual kickoff step.
- External dependencies (upstream teams, vendor feeds, license terms) that constrain what can change.
### 6. Validation & trust
- How was output correctness ever established? Is there a known-good result to check against?
- What does the previous owner *not* trust in their own pipeline?
- Are there silent-failure modes they learned to watch for?
## Method
 
1. **Inventory the artifacts first.** List what you actually have: source files, configs, README/docs, commit history, data dictionaries, notebooks, output samples. You can only judge "answerable from artifacts" if you know what the artifacts are.
2. **Read for the recoverable.** Skim the code and docs enough to answer what *is* answerable. This is what keeps the question list honest — every question you'd have asked but found the answer to is a question you don't waste a human's time on.
3. **Walk the six categories.** For each, generate candidate questions grounded in something specific you saw (a constant, a filter, a table name, a dangling script). Generic questions are weak; anchor every question to an artifact.
4. **Apply the core test.** Drop anything answerable from the repo. Demote anything partially answerable into a sharper "why"-shaped question.
5. **Prioritize.** Rank by risk-if-wrong × likelihood-the-answer-is-lost. A wrong assumption about which data source is authoritative is catastrophic; an unexplained variable name is cosmetic.
## Output format
 
Produce the list grouped by category, ordered with highest-priority questions first within each group. For every question, cite the artifact that prompted it. Use this structure:
 
```markdown
# Questions for [project / previous owner]
 
## What I could answer from the artifacts (so you don't ask these)
- [recoverable fact] — found in `path/to/file:line`
- ...
 
## Open questions for a human
 
### Provenance & authority
1. **[Question]**
   - *Prompted by*: `path/to/artifact` — [the specific thing you saw]
   - *Risk if wrong*: [why this matters]
 
### Decisions without recorded rationale
...
 
[continue through whichever categories yielded questions; omit empty categories]
```
 
Lead with the "already answered" section deliberately — it proves the question list is the residue after honest reading, not a generic checklist, and it spares the previous owner from answering things they already wrote down.
 
## Anti-patterns to avoid
 
- **Inventing rationale.** "The cutoff is probably because the fiscal year ends in March" — no. If it's not recorded, the question is "why this cutoff?", full stop.
- **Generic checklist questions.** "Is there documentation?" is useless. "The `reconcile_weights()` function has no docstring and no caller — is it dead code or a manual step?" is useful.
- **Asking what the repo answers.** Every such question erodes the inheritor's credibility with the person they're interrogating.
- **Treating the list as exhaustive.** Note explicitly that absence of a question in a category means you found no artifact-anchored prompt for one, not that the category is clean.
