# HeyMax plugin

HeyMax helps members plan travel rewards in an AI assistant. The assistant can find award flights and cash fares. It can plan how to fund a trip with airline miles, bank points, Max Miles, and HeyMax First. It can also pick the linked card that earns the most and find merchants where a purchase earns Max Miles.

HeyMax has two parts:

- The HeyMax User MCP server at `https://heymax.ai/user-mcp/mcp`. It holds the tools and the member's data. Each member signs in with their own HeyMax account.
- Four skills in [`skills/`](skills/): `rewards`, `search-flights`, `plan-miles-funding`, and `card-maximiser`. They tell the assistant which tools to call and how to answer.

The skills only plan. Members complete Max Miles transfers and HeyMax First draws in the HeyMax app. The one change the skills make to account data is a recorded loyalty balance, and they save it only when the member asks.

## Use HeyMax in ChatGPT

HeyMax is a plugin in ChatGPT. The plugin includes the server connection and the four skills.

1. Find **HeyMax** in the ChatGPT plugin directory and install it.
2. Click **Connect** and sign in to HeyMax.
3. Review the access on the HeyMax consent screen, then approve it.

## Use HeyMax in other assistants

HeyMax is not listed in other plugin directories yet. Connect the server, then install the skills. HeyMax sign-in works only from the assistants in the table below and the others listed in the [connection guide](docs/mcp.md#connect).

1. Add the remote MCP server `https://heymax.ai/user-mcp/mcp` with the name `heymax-user`.
2. Sign in to HeyMax when the assistant opens your browser.
3. Install the four skills from [`skills/`](skills/). Copy each skill folder into the folder where your assistant loads skills, such as `~/.claude/skills/` for Claude Code. You can also ask the assistant to do it: "Install the HeyMax skills from https://github.com/max-now/heymax-plugin/tree/main/skills."

The assistant needs the server connection to use HeyMax. Without the skills, it can still call the tools, but it does not follow the HeyMax workflows.

| Assistant | Connect the server | Sign in |
| --- | --- | --- |
| Claude Code | `claude mcp add --transport http heymax-user https://heymax.ai/user-mcp/mcp` | Run `/mcp`, select `heymax-user`, and authenticate. |
| Codex | `codex mcp add heymax-user --url https://heymax.ai/user-mcp/mcp` | `codex mcp login heymax-user` |
| Claude web or desktop, Cursor and grok.com | Add a custom connector with the server URL. | Sign in when the app prompts you. |

## Example prompts

- "Find up to three business-class award flights from Singapore to Tokyo for two adults, using my saved balances."
- "How many KrisFlyer miles can I get from my bank points and Max Miles?"
- "Can I afford this 60,000-mile KrisFlyer award, and what would a HeyMax First top-up cost?"
- "Compare award flights and cash fares from Singapore to Hong Kong without booking anything."
- "Find fashion merchants where I can earn Max Miles, and give me the HeyMax link for the one I pick."
- "How much did I spend on my cards this month, and how much bonus cap is left?"
- "Which of my cards should I use at Din Tai Fung?"
- "Remember that my KrisFlyer balance is now 82,500 miles."

Saving a balance changes only the planning balance that HeyMax records. It does not sign in to the airline or bank, and it does not change the balance there.

## Support

Get help from the [HeyMax help centre](https://help.heymax.ai/en/) or at support@heymax.ai. The [privacy policy](https://heymax.ai/company/policy) and the [terms of use](https://heymax.ai/company/terms) cover the plugin.

Read the [User MCP guide](docs/mcp.md) for the connection steps, the access a member grants, and what each tool can do.

HeyMax publishes this repository at each release and does not merge pull requests here.

## License

The public package is licensed under Apache-2.0. The license does not grant
independent rights to use HeyMax trademarks beyond identifying the package.
