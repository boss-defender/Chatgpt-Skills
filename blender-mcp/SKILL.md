---
name: blender-mcp
description: Connect to and work with a user’s Blender application through the official Blender MCP add-on and server, including local Codex access and ChatGPT Work’s remote custom-app path.
metadata:
  short-description: Use Blender through the official MCP integration
---

# Blender MCP

Use this skill for Blender tasks where the user wants an MCP-connected assistant to inspect or modify the open Blender scene.

## Current local setup

- Blender 5.2.1 is installed and satisfies the official Blender MCP requirement of Blender 5.1 or newer.
- The Blender-side add-on must be enabled and running on `localhost:9876`.
- The installed MCP executable is `/home/boss/.local/bin/blender-mcp`.
- The source checkout used for installation is `/tmp/blender_mcp/mcp`; treat it as temporary and do not rely on it for runtime.
- The server supports `stdio` and `http` transports; verify with `blender-mcp --help` when needed.

## Connection architecture

For local Codex access, register the stdio server only when the user authorizes the configuration change:

```sh
codex mcp add blender \
  --env BLENDER_MCP_HOST=localhost \
  --env BLENDER_MCP_PORT=9876 \
  -- blender-mcp
```

Then verify with `codex mcp list` and use a new Codex task/session if the server is not visible in the current tool inventory.

For ChatGPT Work, do not claim that a local stdio server or `localhost` is directly reachable. ChatGPT requires a remote MCP endpoint. Use the official Secure MCP Tunnel for a developer-machine server, expose the HTTP transport through that tunnel, and create a custom MCP app in ChatGPT Work. Developer Mode and workspace admin/owner approval may be required; full write/modify MCP support is plan-dependent.

The shared target architecture is:

```text
Blender add-on (localhost:9876)
        <-> blender-mcp HTTP server
        <-> Secure MCP Tunnel
        <-> ChatGPT Work custom MCP app
```

Codex may use the local stdio registration in parallel with the tunneled ChatGPT Work app. Do not register Codex-only access and describe it as a ChatGPT Work connection.

## Safety and interaction rules

- The official Blender MCP server can execute generated Python in Blender without a complete safety boundary. Recommend a backup or test `.blend` file and avoid sensitive files before enabling write actions.
- Inspect or ask for confirmation before destructive scene edits, file deletion, external uploads, or arbitrary filesystem/network operations.
- If the user intends to edit an existing scene, let them open/select it first and verify the visible scene before modifying it.
- Do not infer that installation or configuration means the MCP tool is callable in the current conversation. Verify the actual tool inventory or report that a new task/session is needed.

## Official references

- Blender MCP page: https://www.blender.org/lab/mcp-server/
- ChatGPT custom MCP apps and Secure MCP Tunnel: https://help.openai.com/en/articles/12584461
