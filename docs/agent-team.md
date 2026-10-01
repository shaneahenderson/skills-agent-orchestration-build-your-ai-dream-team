# Agent team

Mona's Project Pulse dashboard will be built by a coordinated custom-agent team, orchestrated through GitHub Copilot CLI in a Codespace.

| Agent | Target model | Responsibility | Definition |
| --- | --- | --- | --- |
| Orchestrator | Claude Opus 4.7 (copilot) | Breaks the dashboard request into dependency-aware phases, assigns non-overlapping file scopes, coordinates the specialists, and verifies the integrated result. | [`.github/agents/orchestrator.agent.md`](../.github/agents/orchestrator.agent.md) |
| Planner | Claude Opus 4.7 (copilot) | Researches the repository and relevant documentation, identifies requirements and risks, and produces the implementation plan, file assignments, dependencies, edge cases, and validation expectations. | [`.github/agents/planner.agent.md`](../.github/agents/planner.agent.md) |
| Designer | Gemini 3.1 Pro (copilot) | Defines the dashboard's UI/UX, accessibility, information hierarchy, responsive behavior, visual styling, project cards, status badges, and priority treatment. | [`.github/agents/designer.agent.md`](../.github/agents/designer.agent.md) |
| Coder | GPT-5.5 (copilot) | Implements the assigned dashboard logic and runnable-app support, follows the approved design direction, makes errors explicit, and validates the completed code. | [`.github/agents/coder.agent.md`](../.github/agents/coder.agent.md) |

The workflow starts with the Planner's implementation strategy. The Orchestrator turns that strategy into safe phases, coordinating the Designer and Coder in parallel only when their file scopes and dependencies allow it. All agents leave Git staging, commits, and pushes to the learner using Copilot CLI.
