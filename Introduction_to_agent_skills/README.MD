# 1) What are Skills?

## Key takeaways

- Skills are folders of instructions that Claude Code can discover and use to handle tasks more accurately. Each skill
  lives in a SKILL.md file with a name and description in its frontmatter
- Claude uses the description to match skills to requests. When you ask Claude to do something, it compares your request
  against available skill descriptions and activates the ones that match
- Personal skills go in **~/.claude/skills** and follow you across all projects. Project skills go in .claude/skills
  inside a repository and are shared with anyone who clones it
- Skills load on demand — unlike CLAUDE.md (which loads into every conversation) or slash commands (which require
  explicit invocation), skills activate automatically when Claude recognizes the situation
- If you find yourself explaining the same thing to Claude repeatedly, that's a skill waiting to be written

## Where Skills Live

1) **Personal skills** go in **~/.claude/skills** (your home directory). These follow you across all your projects —
   your commit message style, your documentation format, how you like code explained.
2) **Project skills** go in **.claude/skills** inside the root directory of your repository. Anyone who clones the repo
   gets these skills automatically. This is where team standards live, like your company's brand guidelines, preferred
   fonts, and colors for web design.

## Skills vs. CLAUDE.md vs. Slash Commands

1) **CLAUDE.md** files load into every conversation. If you want Claude to always use TypeScript's strict mode, that
   goes in CLAUDE.md.
2) **Skills** load on demand when they match your request. Claude only loads the name and description initially, so they
   don't fill up your entire context window. Your PR review checklist doesn't need to be in context when you're
   debugging — it loads when you actually ask for a review.
3) **Slash commands** require you to explicitly type them. Skills don't. Claude applies them when it recognizes the
   situation.

## When to Use Skills

- Code review standards your team follows
- Commit message formats you prefer
- Brand guidelines for your organization
- Documentation templates for specific types of docs
- Debugging checklists for particular frameworks

# 2) Creating your first skill

## Key takeaways

- A skill is a directory containing a SKILL.md file with metadata (name, description) in frontmatter and instructions
  below
- Claude loads only skill names and descriptions at startup, then matches incoming requests against those descriptions
  using semantic matching
- You get a confirmation prompt before Claude loads the full skill content into context
- Priority for name conflicts: Enterprise → Personal → Project → Plugins
- To update a skill, edit its SKILL.md. To remove one, delete its directory. Always restart Claude Code for changes to
  take effect

## Creating a Skill

The name identifies your skill. The description tells Claude when to use it — this is the matching criteria. Everything
after the second set of dashes is the instructions Claude follows when the skill is activated.

```
---
name: pr-description
description: Writes pull request descriptions. Use when creating a PR, writing a PR, or when the user asks to summarize changes for a pull request.
---

When writing a PR description:

1. Run `git diff main...HEAD` to see all changes on this branch
2. Write a description following this format:

## What
One sentence explaining what this PR does.

## Why
Brief context on why this change is needed

## Changes
- Bullet points of specific changes made
- Group related changes together
- Mention any files deleted or renamed
```

## How Skill Matching Works

When Claude Code starts, it scans four locations for skills but only loads the name and description — not the full
content. This is an important detail.

When you send a request, Claude compares your message against the descriptions of all available skills. For example,
"explain what this function does" would match a skill described as "explain code with visual diagrams" because the
intent overlaps.

Once a match is found, Claude asks you to confirm loading the skill. This confirmation step keeps you aware of what
context Claude is pulling in. After you confirm, Claude reads the complete SKILL.md file and follows its instructions.

## Skill Priority

1) **Enterprise** — managed settings, highest priority
2) **Personal** — your home directory (~/.claude/skills)
3) **Project** — the .claude/skills directory inside a repository
4) **Plugins** — installed plugins, lowest priority

# Configuration and multi-file skills

## Key takeaways

- name and description are required — allowed-tools and model are optional but powerful additions
- A good description answers two questions: What does the skill do? When should Claude use it?
- allowed-tools restricts which tools Claude can use when the skill is active — useful for read-only or
  security-sensitive workflows
- **Progressive disclosure**: keep SKILL.md **under 500 lines** and link to supporting files (references, scripts,
  assets) that Claude reads only when needed
- Scripts execute without loading their contents into context — only the output consumes tokens, keeping context
  efficient

## Skill Metadata Fields

The agent skills open standard supports several fields in the SKILL.md frontmatter. Two are required, and the rest are
optional:

- name (required) — Identifies your skill. Use lowercase letters, numbers, and hyphens only. Maximum 64 characters.
  Should match your directory name.
- description (required) — Tells Claude when to use the skill. Maximum 1,024 characters. This is the most important
  field because Claude uses it for matching.
- allowed-tools (optional) — Restricts which tools Claude can use when the skill is active.
- model (optional) — Specifies which Claude model to use for the skill.

## Writing Effective Descriptions

Be explicit with your instructions. If someone told you "your job is to help with docs," you wouldn't know what to do —
and Claude thinks the same way.

1) What does the skill do?
2) When should Claude use it?

## Restricting Tools with allowed-tools

Sometimes you want a skill that can only read files, not modify them. This is useful for security-sensitive workflows,
read-only tasks, or any situation where you want guardrails.

```
---
name: codebase-onboarding
description: Helps new developers understand the system works.
allowed-tools: Read, Grep, Glob, Bash
model: sonnet
---
```

# Skills vs. other Claude Code features

## Key takeaways

- **CLAUDE.md** loads into every conversation and is best for always-on project standards. Skills load on demand and are
  best for task-specific expertise
- **Subagents** run in isolated execution contexts — use them for delegated work. Skills add knowledge to your current
  conversation
- **Hooks** are event-driven (fire on file saves, tool calls). Skills are request-driven (activate based on what you're
  asking)
- **MCP** servers provide external tools and integrations — a different category entirely from skills
- Each feature handles its own specialty — combine them rather than forcing everything into one approach

## CLAUDE.md vs Skills

CLAUDE.md loads into every conversation, always. If you want Claude to use TypeScript strict mode in your project, put
it in your CLAUDE.md file.

Skills load on demand. When Claude matches a request to a skill, that skill's instructions join the conversation. Your
PR review checklist doesn't need to be in context when you're writing new code — it activates when you ask for a review.

### Use CLAUDE.md for:

- Project-wide standards that always apply
- Constraints like "never modify the database schema"
- Framework preferences and coding style

### Use Skills for:

- Task-specific expertise
- Knowledge that's only relevant sometimes
- Detailed procedures that would clutter every conversation

## Skills vs Subagents

Skills add knowledge to your current conversation. When a skill activates, its instructions join the existing context.

Subagents run in a separate context. They receive a task, work on it independently, and return results. They're isolated
from the main conversation.

### Use Subagents when:

- You want to delegate a task to a separate execution context
- You need different tool access than the main conversation
- You want isolation between delegated work and your main context

### Use Skills when:

- You want to enhance Claude's knowledge for the current task
- The expertise applies throughout a conversation

## Skills vs Hooks

Hooks fire on events. A hook might run a linter every time Claude saves a file, or validate input before certain tool
calls. They're event-driven.

Skills are request-driven. They activate based on what you're asking.

### Use Hooks for:

- Operations that should run on every file save
- Validation before specific tool calls
- Automated side effects of Claude's actions

### Use Skills for:

- Knowledge that informs how Claude handles requests
- Guidelines that affect Claude's reasoning

## Putting It All Together

- **CLAUDE.md** — always-on project standards
- **Skills** — task-specific expertise that loads on demand
- **Hooks** — automated operations triggered by events
- **Subagents** — isolated execution contexts for delegated work
- **MCP servers** — external tools and integrations

Each handles its own specialty. Don't force everything into skills when another option fits better — and you can use
multiple at a time. Skills provide automatic task-specific expertise, CLAUDE.md is for always-on instructions, subagents
run in isolated contexts, hooks fire on events, and MCP provides external tools.

Use skills when you have knowledge that Claude should apply automatically when the topic is relevant, and combine them
with other features for comprehensive customization.

# Sharing skills

## Key takeaways

- Project skills in .claude/skills are shared automatically through Git — anyone who clones the repo gets them
- Plugins let you distribute skills across repositories via marketplaces for broader community use
- Enterprise managed settings deploy skills organization-wide with the highest priority, ideal for mandatory standards
  and compliance
- Subagents don't automatically see your skills — you must explicitly list skills in a custom agent's frontmatter skills
  field
- Built-in agents (Explorer, Plan, Verify) can't access skills at all — only custom subagents defined in .claude/agents
  can

## Committing Skills to Your Repository

- Team coding standards
- Project-specific workflows
- Skills that reference your codebase structure

## Distributing Skills Through Plugins

Plugins are a way to extend Claude Code with custom functionality designed to be shared across teams and projects. In
your plugin project, create a skills directory that follows a similar file structure to the .claude directory — each
skill gets its own folder with a SKILL.md file inside.

After you distribute your plugin to a marketplace, other users can discover and install it into Claude Code for
themselves.

## Enterprise Deployment Through Managed Settings

Administrators can deploy skills organization-wide through managed settings. Enterprise skills take the highest
priority — they override personal, project, and plugin skills with the same name.

The managed settings file supports features like strictKnownMarketplaces to control where plugins can be installed from:

```
"strictKnownMarketplaces": [
  {
    "source": "github",
    "repo": "acme-corp/approved-plugins"
  },
  {
    "source": "npm",
    "package": "@acme-corp/compliance-plugins"
  }
]
```

## Skills and Subagents

Here's something that surprises people: subagents don't automatically see your skills. When you delegate a task to a
subagent, it starts with a fresh, clean context.

- Built-in agents (like Explorer, Plan, and Verify) can't access skills at all
- Custom subagents you define can use skills, but only when you explicitly list them
- Skills are loaded when the subagent starts, not on demand like in the main conversation

```
---
name: frontend-security-accessibility-reviewer
description: "Use this agent when you need to review frontend code for accessibility..."
tools: Bash, Glob, Grep, Read, WebFetch, WebSearch, Skill...
model: sonnet
color: blue
skills: accessibility-audit, performance-check
---
```

### This pattern works really well when:

- You want isolated task delegation with specific expertise
- Different subagents need different skills (frontend reviewer vs. backend reviewer)
- You want to enforce standards in delegated work without relying on prompts

# Troubleshooting skills

## Key takeaways

- Start with the skills validator tool — it catches structural problems before you spend time debugging other things
- If a skill doesn't trigger, the cause is almost always the description — add trigger phrases that match how you
  actually phrase requests
- If a skill doesn't load, check that SKILL.md is inside a named directory (not at the skills root) and the file name is
  exactly SKILL.md
- If the wrong skill gets used, your descriptions are too similar — make them more distinct
- For runtime errors, check dependencies, file permissions (chmod +x), and path separators (use forward slashes
  everywhere)

When skills don't work, the problem usually falls into one of a few categories: the skill doesn't trigger, doesn't load,
has conflicts, or fails at runtime. The good news is that most fixes are pretty straightforward.

## Use the Skills Validator

The first thing to try is the agent skills verifier command. Installation steps vary by operating system, but using **uv
synk** is the easiest way to get it set up quickly.

Once installed, either navigate to your skill directory or run the command from anywhere. The validator will catch
structural problems before you spend time debugging other things.

## Skill Doesn't Trigger

Your skill exists and passes validation, but Claude isn't using it when you expect. The cause is almost always the
description.

Claude uses semantic matching, so your request needs to overlap with the description's meaning. If there's not enough
overlap, no match. Here's what to do:

- Check your description against how you're actually phrasing requests
- Add trigger phrases users would actually say
- Test with variations like "help me profile this," "why is this slow?", "make this faster"
- If any variation fails to trigger, add those keywords to your description

## Skill Doesn't Load

If your skill doesn't appear when you ask Claude "what skills are available," check these structural requirements:

-- The SKILL.md file must be inside a named directory, not at the skills root -- The file name must be exactly
SKILL.md — all caps on "SKILL", lowercase "md"
Run claude --debug to see loading errors. Look for messages mentioning your skill name. Sometimes this alone will point
you straight to the problem.

## Wrong Skill Gets Used

If Claude uses the wrong skill or seems confused between skills, your descriptions are probably too similar. Make them
distinct. Being as specific as possible doesn't just help Claude decide when to use your skill — it also prevents
conflicts with other similar-sounding skills.

## Runtime Errors

The skill loads but fails during execution. A few common causes:

- Missing dependencies: If your skill uses external packages, they must be installed. Add dependency info to your skill
  description so Claude knows what's needed.
- Permission issues: Scripts need execute permission. Run chmod +x on any scripts your skill references.
- Path separators: Use forward slashes everywhere, even on Windows.

## Quick Troubleshooting Checklist

- Not triggering? Improve your description and add trigger phrases.
- Not loading? Check your path, file name, and YAML syntax.
- Wrong skill used? Make descriptions more distinct from each other.
- Being shadowed? Check the priority hierarchy and rename if needed.
- Plugin skills missing? Clear cache and reinstall.
- Runtime failure? Check dependencies, permissions, and paths.