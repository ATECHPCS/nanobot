# fork-patches

Fork-specific patches that must be re-applied after certain upstream merges,
where an upstream refactor silently drops fork functionality.

## 0001-restore-external-mcp-passthrough-sdk.patch

**Apply after merging upstream v0.3.5+ (specifically HKUDS PR #5343,
"refactor: move MCP lifecycle out of AgentLoop").**

That refactor removes the `self._mcp_servers = mcp_servers or {}` assignment
from `AgentLoop.__init__` while keeping the reader in `_get_sdk_mcp_servers()`.
The field then stays `None`, so external MCP servers (stdio/sse, e.g.
QuickBooks and Graphiti) are never attached to the `claude` CLI subprocess —
only the in-process `nanobot` bridge survives, and SDK-path agents silently
lose all external MCP tools after a restart.

The patch repopulates `self._mcp_servers` from `tools_config.mcp_servers` in
`__init__`, right after `self.tools_config` is set. Restores pre-refactor
behavior.

After any upstream merge, sanity-check with:

    grep -n 'self\._mcp_servers' nanobot/agent/loop.py

There must be an assignment (`self._mcp_servers = ...`), not just the
annotation and the reader.
