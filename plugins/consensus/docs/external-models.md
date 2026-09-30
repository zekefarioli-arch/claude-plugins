# Adding non-Claude models (optional)

The plugin works with Claude subagents only. That already removes most of the "yes to everything" effect, because the stances are forced. What it cannot remove is blind spots that all Claude models share. For that, add a second model family through an MCP server.

## Why this is not bundled in the plugin

- It needs API keys (Gemini, OpenAI, OpenRouter, xAI). Keys do not belong in a plugin repo.
- It needs Python and `uv` on the machine. A plugin that fails to start its MCP server is worse than one without it.
- It costs money per call. It should be a conscious opt in.

## Recommended: PAL MCP

PAL (Provider Abstraction Layer, formerly Zen MCP) ships a `consensus` tool with stance steering, plus a `challenge` tool aimed at reflexive agreement.

Install it at **user scope**, outside this plugin, following the official README:
https://github.com/BeehiveInnovations/pal-mcp-server

To keep context usage low, leave only the tools you need enabled through its `DISABLED_TOOLS` setting. For this plugin, `consensus` and `challenge` are the useful ones.

## How the deliberate skill uses it

When the session exposes a multi-model consensus tool, step 5 of the skill sends the same brief to it with one model "for" and one "against", and the synthesis treats those answers as extra voices. When no such tool exists, step 5 is skipped silently.

A cheap setup that still adds real diversity: one OpenRouter key, and two models from different vendors.
