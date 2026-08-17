# OpenHands Catalog

This repo is the shared toolbox for our OpenHands agents. If you want the agent
to know how to do something new, run a specific tool, follow a team
convention, call an internal service; you add it here. You don't touch any
agent's code, and nobody has to rebuild a Docker image.

Every agent on the team points at this repo (pinned to a release tag) and
picks up whatever's published here.

## Contributing something. The short version

1. Make a folder under `plugins/` for your thing.
2. Add one required file: `.plugin/plugin.json`.
3. Add an entry for it in `.plugin/marketplace.json`.
4. Open a PR.
5. Once merged, tag a release.

No agent code changes, no image rebuild and no redeploy. Agents move
to your new plugin whenever *they* bump the tag they're pinned to, it never
happens automatically underneath them.

## Step by step: adding a plugin

Say you want to add a plugin called `db-tools`.

**1. Create the folder structure**

```
plugins/db-tools/
├── .plugin/
│   └── plugin.json
└── skills/
    └── db-tools/
        └── SKILL.md
```

Only `plugin.json` is required. Everything else, `skills/`, `agents/`,
`commands/`, `hooks/hooks.json`, `.mcp.json` is optional, and you can add
it later without restructuring anything.

**2. Write `plugin.json`**

```json
{
  "name": "db-tools",
  "version": "1.0.0",
  "description": "Helpers for querying and migrating our internal databases",
  "author": {
    "name": "Your Name",
    "email": "you@company.com"
  }
}
```

Only `name` is required. The rest have sensible defaults. `license` and
`repository` are also fine to add if relevant; extra fields don't break
anything.

**3. Write your skill**

`skills/db-tools/SKILL.md` is a normal skill file instructions in Markdown
that tell the agent when and how to use this capability. If you've written a
skill before, this part is identical.

**4. (Optional) Give it an MCP server**

If your plugin needs to talk to a live tool/service (not just shell out to a
CLI), add `.mcp.json` at the plugin root:

```json
{
  "mcpServers": {
    "db-tools": {
      "command": "uvx",
      "args": ["mcp-server-postgres"]
    }
  }
}
```

> **Important:** `.mcp.json` only does anything when it's at the **plugin
> root**, next to `plugin.json`. It does nothing useful inside a bare skill
> folder that isn't part of a plugin. The agent will never connect to that
> server. If your addition needs an MCP server, it must be a plugin, not a
> standalone skill.

**5. Register it in the marketplace index**

Open `.plugin/marketplace.json` at the repo root and add your plugin to the
`plugins` list:

```json
{
  "name": "our-team-catalog",
  "owner": { "name": "Our Team", "email": "team@company.com" },
  "description": "Shared plugins and skills for our OpenHands agents",
  "plugins": [
    {
      "name": "db-tools",
      "source": "./plugins/db-tools",
      "description": "Helpers for querying and migrating our internal databases"
    }
  ],
  "skills": []
}
```

This file is the index. It's how agents discover that `db-tools` exists at
all. Forgetting this step is the most common way a new plugin "silently
doesn't show up."

**6. Open a PR**

Normal review process. Nothing agent-side needs to change for your PR to be
mergeable.

## Releasing a new version

Agents don't automatically get new plugins the moment they're merged to
`main` . They're pinned to a git tag on purpose, so a working setup never
breaks under someone's feet. To actually ship what's in `main`:

```bash
git checkout main
git pull
git tag v0.2.0
git push origin v0.2.0
```

Pick the next version number up from whatever the last tag was. A tag is just a tag, like any other git repo.

Anyone who wants your new plugin now updates the version their agent points
at (usually one line, e.g. `CATALOG_REF = "v0.2.0"`) and redeploys/restarts
their agent. Until they do that, they keep working exactly as before.

## Repo layout

```
openhands-catalog/
├── .plugin/
│   └── marketplace.json      # the index — lists every plugin & skill
└── plugins/
    └── <plugin-name>/
        ├── .plugin/
        │   └── plugin.json   # required: name, version, description, author
        ├── .mcp.json          # optional: MCP servers this plugin provides
        ├── skills/            # optional: one or more SKILL.md files
        ├── agents/            # optional: subagent definitions
        ├── commands/          # optional: slash commands
        └── hooks/
            └── hooks.json     # optional: lifecycle hooks
```

## FAQ

**Do I have to fill in every field?**
No. `plugin.json` only requires `name`. Everything in the folder structure
above besides `plugin.json` itself is optional.

**Can I just add a skill without making it a full plugin?**
Yes — drop it under a `skills:` entry in `marketplace.json` instead of
`plugins:`, pointing at a folder with a `SKILL.md`. Do this if your addition
is pure instructions with no MCP server, hooks, subagents, or slash commands
attached. The moment you need any of those, make it a plugin instead.

**I added `.mcp.json` to my skill folder and it's not working.**
Expected — see the callout above. Move your skill into a plugin folder
structure; `.mcp.json` is only honored at the plugin root.

**How do I know what tag an agent is currently using?**
Ask whoever owns that agent, or check its config for a `CATALOG_REF` (or
similarly named) constant. That's the pinned version it's running against.
