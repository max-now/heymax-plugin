# HeyMax plugin

This repository contains the cross-platform HeyMax travel-rewards plugin for the authenticated HeyMax User MCP. It helps travelers find award flights, compare cash fares, and plan how to fund trips with airline miles, bank points, Max Miles, and HeyMax First capacity, and get the most from their linked cards, with supporting rewards and earning tools. One set of skills (`rewards`, `search-award-flights`, `plan-miles-funding`, and `card-maximiser`) ships to ChatGPT, Codex, Claude, Muse, Grok, and Cursor, each through its own manifest, and every manifest points to the same production MCP server.

The backend remains the source of truth for OAuth, user isolation, tool schemas, rate limits, business logic, and deployment. This repository contains no backend code, client secret, access token, or OAuth proxy. The skills rely on the tools and schemas returned by MCP discovery for each session; no duplicate tool catalog is shipped here. Most tools are read-only; the loyalty-balance setter is the one account-data write the skills perform. Max Miles transfers and HeyMax First draws are planned in chat and completed in the HeyMax app, because directory policies prohibit executing money movement and collecting one-time codes.

## Installation and authentication

Installing the plugin registers the remote MCP server `heymax-user` at `https://heymax.ai/user-mcp/mcp`; users never paste the URL. Authentication is a separate per-user OAuth step, and the HeyMax consent screen lets each user choose which permissions to grant.

| Client | How users install | How users authenticate |
| --- | --- | --- |
| ChatGPT and Codex | Install HeyMax from the plugin directory once published. | Click **Connect** and sign in to HeyMax. |
| Claude web, desktop, mobile, and Cowork | Install the HeyMax plugin or connector from the directory once published. | Connect `heymax-user` from the plugin's **Connectors** tab. |
| Claude Code | `/plugin marketplace add max-now/heymax-plugin`, then `/plugin install heymax@heymax`. | Open `/mcp`, select `plugin:heymax:heymax-user`, and complete OAuth. |
| Muse Code | `muse plugins marketplace add heymax https://github.com/max-now/heymax-plugin`, `muse plugins install heymax@heymax`, then `muse plugins approve heymax`. | `muse mcp login heymax-user` (Muse Code 1.4.11 or later). |
| Grok Build | `grok plugin install heymax@xai-official` once listed, or install from this repository's Claude marketplace. | The browser opens on the first HeyMax tool call. |
| Grok Bot and Cursor | Install from the marketplace once listed. | Connect HeyMax when prompted. |
| grok.com | Add a custom connector with the MCP URL. | Sign in when Grok prompts. |

## Example prompts

- “Find up to three business-class award flights from Singapore to Tokyo for two adults, using my saved balances.”
- “How many KrisFlyer miles can I get from my bank points and Max Miles?”
- “Can I afford this 60,000-mile KrisFlyer award, and what would a HeyMax First top-up cost?”
- “Compare award flights and cash fares from Singapore to Hong Kong without booking anything.”
- “Find fashion merchants where I can earn Max Miles, and give me the HeyMax link for the one I pick.”
- “How much did I spend on my cards this month, and how much bonus cap is left?”
- “Which of my cards should I use at Din Tai Fung?”
- “Remember that my KrisFlyer balance is now 82,500 miles.”

The balance setter updates only HeyMax's recorded planning balance. It does not log in to or modify the airline or bank account. The skills call it only for an explicitly requested absolute current balance; they never infer a write from a search, expected transfer, or expected booking.

## Support

Get help from the [HeyMax help centre](https://help.heymax.ai/en/) or at support@heymax.ai. The [privacy policy](https://heymax.ai/company/policy) and the [terms of use](https://heymax.ai/company/terms) cover the plugin.

HeyMax publishes this repository at each release and does not merge pull requests here.

## License

The public package is licensed under Apache-2.0. The license does not grant
independent rights to use HeyMax trademarks beyond identifying the package.
