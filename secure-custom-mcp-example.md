# Secure Custom MCP Example

## The Problem

Some third-party providers only issue one broad API key. That key may be able to:

- Read customers
- Update customers
- Export data
- Trigger refunds
- Change subscriptions
- Delete records

A support AI assistant does not need all of that access. It only needs to help the support team read customer context and prepare useful responses.

## Put a Custom MCP in the Middle

Instead of exposing the provider API directly to the AI assistant, place a custom MCP between them:

```text
AI Assistant → Custom MCP → Provider API
```

The provider API key stays on the server where the MCP runs. The AI assistant only sees the tools the MCP chooses to expose.

## Expose Safe Business Capabilities

The MCP should expose small, business-specific tools such as:

```text
read_customer_summary
list_recent_tickets
get_invoice_status
draft_support_response
prepare_billing_change
```

These tools describe what the support assistant is allowed to do, not everything the provider API can do.

## Do Not Expose Dangerous Operations

The MCP should deliberately avoid exposing risky tools such as:

```text
delete_customer
refund_payment
export_all_customers
update_subscription
change_permissions
```

If the MCP does not expose these operations, the AI assistant cannot call them through the MCP.

## Simplified TypeScript Example

```ts
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

const server = new McpServer({
  name: "secure_support_mcp",
  version: "1.0.0",
});

const PROVIDER_API_KEY = process.env.PROVIDER_API_KEY;

server.tool(
  "read_customer_summary",
  {
    email: z.string().email(),
  },
  async ({ email }) => {
    if (!email.endsWith("@customer-domain.com")) {
      throw new Error("Customer is outside the allowed scope.");
    }

    const response = await fetch(
      `https://provider.example.com/customers?email=${encodeURIComponent(email)}`,
      {
        headers: {
          Authorization: `Bearer ${PROVIDER_API_KEY}`,
        },
      }
    );

    if (!response.ok) {
      throw new Error("Failed to fetch customer.");
    }

    const customer = await response.json();

    const safeSummary = {
      name: customer.name,
      plan: customer.plan,
      status: customer.status,
      recent_ticket_count: customer.recent_ticket_count,
      invoice_status: customer.invoice_status,
    };

    console.log({
      action: "read_customer_summary",
      email,
      returned_fields: Object.keys(safeSummary),
      timestamp: new Date().toISOString(),
    });

    return {
      content: [
        {
          type: "text",
          text: JSON.stringify(safeSummary, null, 2),
        },
      ],
    };
  }
);
```

## The Boundary Matters Most

The important part is not the code itself. The important part is the boundary.

The MCP:

```text
Validates the request
Keeps the provider API key hidden
Calls the provider from the server
Filters the response
Returns only safe fields
Logs the action
Blocks risky operations by not exposing them
```

## Design Principle

Do not expose the provider API to the AI.

Expose the smallest useful business capability.

A weak design asks:

```text
Can the AI connect to this system?
```

A better design asks:

```text
What should the AI be allowed to do?
```

Custom MCPs are useful because they let you create granular capabilities, even when the underlying provider does not offer granular API keys.
