# Implementing Figma designs with GitHub Copilot and MCP servers demo

A sandbox repo for evaluating and comparing two Figma MCP (Model Context Protocol) servers that let AI coding agents translate Figma designs into production-ready code directly inside VS Code using GitHub Copilot.

## Supported MCP Servers

This project ships with configurations for **two** Figma MCP servers (see [.vscode/mcp.json](.vscode/mcp.json)):

| Server | Type | Source |
|---|---|---|
| **Figma MCP Server (Official)** – `com.figma.mcp/mcp` | Remote (HTTP) | [Figma MCP Server Guide](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server) |
| **Framelink MCP for Figma** | Local (stdio via `npx`) | [figma-developer-mcp on npm](https://www.npmjs.com/package/figma-developer-mcp) · [GitHub repo](https://github.com/GLips/Figma-Context-MCP) |

> **Important:** Enable only **one** server at a time to avoid confusing the AI agent. Disable the other in GitHub Copilot "Configure tools..." dialogue before starting a session.

## Getting Started

### Prerequisites

- [VS Code](https://code.visualstudio.com/) with [GitHub Copilot](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) extension
- A [Figma](https://www.figma.com/) account and a personal access token (for Framelink) or OAuth (for the official server)
- For the official Figma MCP Server: Full or Dev seat license (free accounts only have 6 MCP calls per month). For using Code Connect feature: Organization or Enterprise team.
- Node.js ≥ 18

### Setup

1. **Clone the repo**

   ```bash
   git clone <repo-url>
   cd figma-implement-design-copilot
   ```

2. **Configure the MCP server**

   Open [.vscode/mcp.json](.vscode/mcp.json) and keep only the server you want to test enabled. If using Framelink, replace `YOUR_FIGMA_TOKEN` with your personal access token.

3. **Fine-tune Copilot custom instructions**

   Review [.github/copilot-instructions.md](.github/copilot-instructions.md) and adjust the rules to match your project's framework, styling approach, and conventions. These instructions are loaded automatically on every Copilot Chat session and control how the agent interacts with the Figma MCP server — including the order of tool calls, how assets are handled, and which design-system components to prefer. Tailoring them to your stack (e.g., switching from React + Tailwind to Vue + CSS Modules) ensures the generated code fits your project from the start.

4. **Generate project rules**

   Run the saved prompt `/figma-instructions` (defined in [.github/prompts/figma-instructions.prompt.md](.github/prompts/figma-instructions.prompt.md)) in Copilot Chat. This analyzes the codebase and generates a detailed [AGENTS.md](AGENTS.md) file containing token definitions, component conventions, and styling rules that guide the AI agent.

5. **Implement a design**

   Paste a Figma URL into Copilot Chat:

   ```
   implement this design https://www.figma.com/design/XXXXX/...
   ```

   The agent will use the active MCP server to fetch design context, screenshots, and assets, then generate code following the project's conventions.

## Project Structure

```
figma-demo/
├── .github/
│   ├── copilot-instructions.md   # Copilot custom instructions
│   ├── prompts/
│   │   └── figma-instructions.prompt.md  # Saved prompt to generate AGENTS.md
│   └── skills/
│       ├── code-connect-components/      # Skill: connect Figma ↔ code components
│       └── implement-design/             # Skill: translate Figma → code
├── .vscode/
│   └── mcp.json                  # MCP server configurations
├── AGENTS-outputexample.md       # Sample of the auto-generated rules for Figma-to-code workflow
└── README.md
```

Once you start implementing designs, the project will grow with `src/`, `tailwind.config.js`, `package.json`, etc.

## How It Works

1. **Copilot custom instructions** ([.github/copilot-instructions.md](.github/copilot-instructions.md)) define the Figma MCP integration workflow — fetching design context, screenshots, and assets before writing code.
2. **AGENTS.md** captures project-specific conventions (color tokens, typography, component patterns, spacing) so the agent produces code that fits the existing codebase.
3. **Skills** (in `.github/skills/`) provide specialized agent behaviors for design implementation and Code Connect mapping.

> **⚠️ Avoid conflicts between instruction files.** Both `copilot-instructions.md` and `AGENTS.md` are loaded together during design implementation. If they contain ambiguous or contradictory statements (e.g., different styling rules or conflicting component patterns), the agent may produce inconsistent output. Keep `copilot-instructions.md` focused on the **workflow** (tool call order, asset handling, general rules) and `AGENTS.md` focused on **project-specific conventions** (tokens, components, file paths). Review both files after regenerating `AGENTS.md` to ensure they are aligned.

## Code Connect

If your Figma plan supports it (Organization or Enterprise), **enable [Code Connect](https://www.figma.com/developers/code-connect)**. Code Connect links your actual code components to their Figma counterparts, so when the MCP server fetches design context it returns references to your real components instead of raw layout descriptions. This dramatically improves output quality:

- **Higher fidelity** — the agent reuses your existing components with correct prop signatures instead of recreating them from scratch.
- **Less manual cleanup** — generated code already imports and composes your design-system components.
- **Consistent token usage** — mapped components carry your project's tokens (colors, spacing, typography), reducing arbitrary values.

This project includes a `code-connect-components` skill (`.github/skills/code-connect-components/`) to help set up Code Connect mappings.

## Useful Links

- [Guide to the Figma MCP Server](https://help.figma.com/hc/en-us/articles/32132100833559-Guide-to-the-Figma-MCP-server) — Official Figma documentation on the MCP server
- [figma/mcp-server-guide](https://github.com/figma/mcp-server-guide) — A guide on how to use the Figma MCP server (GitHub)
- [Code with Context using Figma MCP Server and GitHub Copilot](https://www.figma.com/webinars/mcp-github-copilot/) — Webinar recording
- [Framelink MCP for Figma (GitHub)](https://github.com/GLips/Figma-Context-MCP) — Open-source alternative MCP server
- [Browse Figma-related skills on skills.sh](https://skills.sh/?q=figma) — Discover and install additional agent skills for Figma workflows

## Tips

- **Use an advanced model.** Select a premium/advanced model (e.g., Claude Sonnet 4, GPT-4.1) in the Copilot Chat model picker before running prompts. Design-to-code tasks involve complex multi-step reasoning, large context windows, and nuanced visual interpretation — advanced models produce significantly better results than smaller ones.
- **One server at a time.** The two MCP servers expose similar tool names. Running both simultaneously will confuse the agent.
- **Run `/figma-instructions` first.** This generates `AGENTS.md` with rules tailored to your codebase, significantly improving output quality.
- **Provide specific Figma URLs.** Link to the exact frame or component node for best results — full-page URLs may produce truncated context.
- **Iterate visually.** After the agent generates code, compare the rendered output against the Figma screenshot and ask for adjustments.

## License

MIT