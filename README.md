# claude-plugins

Personal Claude Code plugin marketplace.

## Plugins

| plugin | what it does |
| :- | :- |
| `consensus` | Structured deliberation for design decisions. Three subagents with forced stances (advocate, critic, pragmatist) argue independently, rebut each other once, and a synthesis keeps disagreements visible instead of averaging them away. |

## Layout

```
claude-plugins/
├── .claude-plugin/marketplace.json     # marketplace catalog
└── plugins/consensus/
    ├── .claude-plugin/plugin.json      # plugin manifest
    ├── skills/deliberate/SKILL.md      # orchestration: frame, fan out, rebut, synthesize
    ├── agents/
    │   ├── advocate.md                 # strongest honest case for one option
    │   ├── critic.md                   # forced strongest objection (runs on opus)
    │   └── pragmatist.md               # cost, time to value, ops, reversibility
    └── docs/external-models.md         # optional: add non-Claude models via PAL
```

## Install

From GitHub (after pushing this repo):

```
/plugin marketplace add zekefarioli-arch/claude-plugins
/plugin install consensus@zeke-plugins
```

Local development, without installing:

```
claude --plugin-dir ./plugins/consensus
```

Validate after any change:

```
claude plugin validate ./plugins/consensus
claude plugin validate .
```
## Updating

Claude Code pins an installed plugin to the `version` in its `plugin.json`. Pushing changes without bumping it means nobody, including you, receives them.

1. Bump `version` in `plugins/consensus/.claude-plugin/plugin.json` (for example `0.1.0` to `0.1.1`).
2. Validate: `claude plugin validate ./plugins/consensus`
3. Commit and push.
4. In Claude Code: `/plugin marketplace update zeke-plugins`

## Usage

```
/consensus:deliberate Should the routing engine keep per-card utilisation in ETS or read it from Postgres on every authorisation?
/consensus:deliberate quick Oban unique jobs or a Postgres advisory lock for the TrueLayer sync?
```

Claude can also trigger it on its own when you ask for a debate, a second opinion or a devil's advocate on a design decision.

## Design notes

- **Independence first.** Round 1 runs the three subagents in parallel from a written brief. They never see the conversation or each other, so they cannot anchor on the user's framing or on each other.
- **Forced stances beat voting.** Models trained on similar data tend to agree. The critic must produce the strongest objection even when it likes the option; that is what breaks reflexive agreement.
- **Locked decisions are respected.** The skill reads `CLAUDE.md` and ADR folders and refuses to reopen settled decisions unless asked.
- **Read only agents.** Subagents get `Read`, `Grep` and `Glob` so they can cite the codebase but cannot change it.
- **Model mix.** The critic runs on `opus`; the others inherit the session model. Change the `model:` field in each agent to tune cost.
