# Claude Code 101

## Claude Code Useful Commands

1. **/context** — visualizes your current context window usage as a colored grid. Shows a colored grid of context
   usage + optimization suggestions.
2. **/compact** — compacts everything up to that point. Handy when you want to free up context space while keeping a
   memory of what you previously worked on.
3. **/clear** — if you want to completely start from scratch with no memory of the previous session. This removes
   everything.
4. **/btw** — ask a quick side question without interrupting Claude's current work.
5. **/init** — creates a CLAUDE.md file with codebase documentation, project structure, build/test commands, and other
   key details for future Claude Code sessions.
6. **/agents** — opens the agent management interface, where you can:
    - View available built-in and custom agents (like Explore, Plan, claude-code-guide, etc.)
    - Create custom agents with specific system prompts, tools, and models tailored to a task
    - Edit or delete existing custom agents
7. **/mcp** — manage MCP (Model Context Protocol) servers — external integrations that extend Claude's capabilities with
   additional tools and data sources.
8. **/plugins** — Opens the plugin manager: browse, install, enable/disable, and remove plugins, and add marketplaces
   (repositories of plugins). Used to hook up custom    
   slash commands, agents, hooks, and MCP servers.
9. Claude --debug — Runs Claude Code with verbose debug logging. Surfaces internal details like tool calls, MCP server
   activity, API requests, and errors. Useful for  
   troubleshooting. You can scope the output by category, e.g. claude --debug api or claude --debug hooks.

## Claude Code on the Web

Claude Code on the web is a solid option if you want to remotely work on projects through a GitHub repository.

## Plan Mode

Within the Shift + Tab menu is Plan Mode. Plan Mode takes your prompt and uses read-only tools to analyze your codebase
and research your suggested implementation. It will ask clarifying questions along the way, then return a detailed plan
it can execute.

Plan Mode is great for planning complex changes or doing a safe code review. Many times you'll be asking Claude to
handle multi-step implementations toward a feature, and this is exactly where Plan Mode excels.

## The Explore → Plan → Code → Commit Workflow

If you take one thing away from this course, let it be this workflow: **Explore, Plan, Code, and Commit**. Without it,
most people jump straight to asking Claude to write code — which means more course-correcting later on.

- **Add tools.** Tools that help Claude complete its goals remove a lot of back and forth. For example, if you're
  building web UIs, install the Claude in Chrome extension so Claude Code can control a browser tab and test the UI
  directly.

- **Quick tip:** If you find Claude keeps running into the same issues, ask it to save the solution to its CLAUDE.md
  file.

- **Commit.** Once you've tested the changes yourself and are happy with the results, it's time to push your code.
  Before you commit, run a subagent code reviewer to look at your work. A subagent gets a fresh pair of eyes on the
  codebase — it doesn't carry the bias the main agent might have from the session.

## Context Management

Use subagents. Subagents run in parallel with your main agent but have a completely separate context window. For tasks
where you only need the answer — like "where are the authentication endpoints located?" — a subagent does the work and
returns just a summary to your main agent, keeping your primary context clean.

## Code Review

Claude Code has a few built-in features that make your git workflow faster. Let's go through them.

**Review with a Subagent**

Before you push a PR, ask Claude to use a subagent to review your changes. The subagent runs in its own context window
with fresh eyes — it doesn't carry the bias of the main agent that just spent the session writing the code.

**Recap**

Use a subagent for an unbiased code review before pushing. Use `/commit-push-pr` to handle the full commit-to-PR flow in
one step. And use `--from-pr` to resume work on a PR later. These are small features, but they remove a lot of friction
from your daily workflow.

## CLAUDE.md File

One of the most useful features in Claude Code is the CLAUDE.md file. It gives Claude Code persistent memory about your
project.

- **Project-level CLAUDE.md** lives in the root directory of your project. Shared with the team.

- **User-level CLAUDE.md** lives in your configuration folder. This one is just for you and applies across all your
  projects. Put your personal preferences here.

- **Reference project docs.** If you have documentation in your project that you want Claude to reference, use the `@`
  symbol with the file path:

    - README.md — please read if you need more info: `@README.md`

Scopes:

1. Project — for the whole team: `project/.CLAUDE.md`
2. User — just for you: `~/.CLAUDE.MD`

## Subagents

Claude can delegate tasks to subagents that break them down and run component tasks in parallel, improving your context
management. Each subagent operates in its own isolated context window.

This is where subagents come in. Claude spawns a subagent to handle a task like "explore this codebase for me." The
subagent runs in parallel with its own context window, does all the exploration work, and once finished, summarizes its
findings and returns that summary back to Claude.

**Further Customization**

- **Persistent memory** lets your subagent retain memory across conversations. This is great if you're using it
  consistently on the same projects.

- **Preload skills** into subagents by adding the skill key and listing skills by name. Note that unlike skills in your
  main conversation, the entire skill is loaded into context here.

**Recap**

Keeping your context window clean is one of the best ways to stay productive with Claude Code. With subagents, you can
run an agent in the background to handle the heavy lifting and return just the answer to your main context window.

- **General-purpose** subagent — use for multi-step tasks.
- **Explore** subagent — used for fast searching of codebases.
- **Plan** subagent — used during Plan Mode for research and analysis of your codebase before presenting a plan.

> Want to go deeper? Check out our dedicated course: https://anthropic.skilljar.com/introduction-to-subagents

## Skills

> Want to go deeper? Check out our dedicated course: https://anthropic.skilljar.com/introduction-to-agent-skills

## MCP

Model Context Protocol (MCP) is an open standard that lets Claude Code connect to external tools and data sources. When
you ask a question, Claude automatically understands when it should use those tools to better handle your query.

A lot of your context lives outside your codebase — in databases, productivity apps, or public repositories. MCP bridges
that gap.

**What Can You Do With It?**

For example, if your team uses Linear for project management, you can add a Linear MCP server to bring in the details of
your specific issues. If you need up-to-date documentation for a dependency, a docs MCP server like Context7 can provide
that to Claude Code.

**Scoping Servers**

MCP servers can be scoped in three ways:

- **Local** — only available in the current project, just for you.
- **User** — available across all your projects.
- **Project** — uses a `.mcp.json` file that you check into version control, so your team gets it automatically.

If your MCP tools exceed 10% of your context window, Claude Code automatically switches to tool search mode, which
discovers the right tools on demand — though this may not work as reliably.

**Recap**

MCP connects Claude Code to your external tools and data sources. Add servers with `claude mcp add`. Scope them to your
project with `.mcp.json` so your team gets them automatically. And keep an eye on context usage by disabling servers
you're not actively using.

## Hooks

Hooks let you run commands at specific points in Claude Code's lifecycle. The key difference between hooks and
everything else covered in this course is that hooks are deterministic — they always run.

**Why Use Hooks**

You can tell Claude in your CLAUDE.md to run Prettier after every file edit. Most of the time it will. But sometimes it
won't. A hook makes it happen every single time, no exceptions.

Common use cases include:

- Auto-formatting after file edits
- Logging all executed commands for compliance
- Blocking dangerous operations like modifying production files
- Sending yourself notifications when Claude finishes a task

**How They Work**

Hooks are configured in your settings.json. You pick an event, optionally set a matcher for which tools it applies to,
and provide a command to run. The available events are:

- **PreToolUse** — runs before a tool call
- **PostToolUse** — runs after a tool call completes
- **UserPromptSubmit** — runs when you submit a prompt, before Claude processes it
- **Stop** — runs when Claude finishes responding
- **Notification** — runs when Claude sends a notification

You configure them through the `/hooks` command inside Claude Code, or by editing settings.json directly.

**Blocking with PreToolUse**

PreToolUse hooks can block tool calls before they execute. Your hook receives the tool name and input as JSON on stdin.
The exit code determines the behavior:

- **Exit code 0** — proceed normally.
- **Exit code 2** — block the action. The stderr message gets fed back to Claude as feedback so it knows why it was
  blocked and can adjust.
- **Any other exit code** — a non-blocking error that gets shown to you but doesn't stop anything.

Hooks are configured in `.claude/settings.json`.

Use the `CLAUDE_PROJECT_DIR` environment variable in your commands to reference scripts stored in your project, so they
work regardless of Claude's current working directory.

**Recap**

Hooks give you deterministic control over Claude Code's behavior. Use PostToolUse for auto-formatting and logging. Use
PreToolUse to block dangerous operations. Configure them with `/hooks` or in settings.json. And check them into your
repo so your team gets them too.

If something needs to happen every time without fail, don't put it in a prompt. Put it in a hook.
