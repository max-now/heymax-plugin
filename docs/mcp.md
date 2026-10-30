# Connect HeyMax to an AI assistant

HeyMax's User MCP lets a member use their own rewards and travel-planning data in an AI assistant. It is available to HeyMax members in Singapore and Hong Kong. Linked-card spend analysis and HeyMax First are available only in Singapore.

## Connect

Install the official HeyMax plugin or connector in your assistant, choose **Connect**, and sign in to HeyMax. Review the access described on the HeyMax consent screen before approving. You can disconnect or reconnect in the assistant's connector settings.

Clients that accept a custom remote MCP server can use:

```text
https://heymax.ai/user-mcp/mcp
```

The server uses per-member OAuth. MCP clients should follow the server's authorization discovery. The connection grants access to the public tools listed in the tool reference for the signed-in member. Do not ask a member to paste a password, access token, or one-time code into a chat.

## What the connector does

| Capability | Access | Effect |
| --- | --- | --- |
| Rewards, loyalty accounts, linked cards, card spend, merchant offers, and transfer quotes | Read | Returns information available to the signed-in member. Quotes are plans; they do not move points or miles. |
| Cached award and cash flight search | Read | Returns search observations for comparison. Prices and availability can change. |
| Live award search | Write | Starts a private search task and returns results when ready. It does not hold, book, or buy a seat. |
| Recorded loyalty balance | Write | Replaces one balance recorded in the member's private HeyMax planning data, only after the member explicitly asks to save an absolute current balance. It does not change the balance at a bank or airline. |

Supported clients may render flight comparisons and merchant offers as interactive cards. A merchant link supplied by HeyMax may be a tracked link; the assistant should identify it as such. Any purchase takes place on the merchant's site.

The connector does not execute Max Miles transfers or HeyMax First draws. Members complete those actions in the HeyMax app. It does not request one-time codes in chat.

The [tool reference](mcp-tools.md) describes each tool's inputs, results, and effects. The live MCP `tools/list` response is the source of truth for the current schemas and availability.

## Examples

- "Find up to three business-class award flights from Singapore to Tokyo for two adults, using my saved balances."
- "How many KrisFlyer miles can I get from my bank points and Max Miles?"
- "Which of my linked cards should I use at Din Tai Fung, and how much bonus cap is left?"
- "Remember that my KrisFlyer balance is now 82,500 miles."

## Data and support

While connected, the assistant can access the public HeyMax tools for the signed-in member. The member can disconnect in the assistant's connector settings. The [privacy policy](https://heymax.ai/company/policy) explains data handling, and the [terms of use](https://heymax.ai/company/terms) cover the service.

For help, visit the [HeyMax help centre](https://help.heymax.ai/en/) or email support@heymax.ai.
