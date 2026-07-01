# matrix-react-native-dev-plugin

Claude Code plugin encoding Ono Apps' React Native SDLC workflow.

## Pipeline

| Stage | Command | Skill | Agent(s) |
|---|---|---|---|
| 1. Analyze | `/analyze-feature` | `rn-repo-analysis` | `repo-analyst`, `rn-architect` |
| 2. Plan | `/create-dev-plan` | `rn-dev-planning` | `rn-architect` |
| 3. Implement | `/implement-task` | `rn-feature-implementation` | `rn-feature-developer` |
| 4. Review | `/review-code` | `rn-code-review` | `rn-code-reviewer`, `rn-performance-reviewer` |
| 5. Fix | `/fix-review-comments` | `rn-debugging` | `rn-debugger`, `rn-feature-developer` |
| 6. QA handoff | `/create-dev-qa-notes` | `rn-testing-and-qa-handoff` | `rn-feature-developer` |
| 7. Release | `/prepare-mobile-release` | `rn-release-readiness` | `rn-release-engineer`, `rn-performance-reviewer` |

`repo-analyst` has no dedicated command — it's invoked by other agents/skills as a first step.

## Status

Scaffolding only. Standards, templates, agents, skills, commands, and hooks are placeholders — content is being filled in incrementally.
