# AI Examples

A small collection of practical examples for building safer, more useful AI systems.

## Examples

### Secure Custom MCP Example

[`secure-custom-mcp-example.md`](./secure-custom-mcp-example.md) shows how to place a custom Model Context Protocol (MCP) server between an AI assistant and a third-party provider API.

The example demonstrates how to:

- Keep a broad provider API key hidden from the AI assistant.
- Expose only safe, business-specific capabilities.
- Validate requests before calling the provider.
- Filter provider responses down to safe fields.
- Avoid exposing destructive actions such as deleting customers, issuing refunds, or exporting all customer data.

## Core Principle

Do not expose powerful provider APIs directly to an AI assistant.

Expose the smallest useful business capability instead.

## Who This Is For

These examples are useful for developers designing AI assistants, support workflows, internal tools, or MCP integrations where safety and scoped access matter.
