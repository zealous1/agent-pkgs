# 0019. An upstream bin/ ships beside the interpreter links

Date: 2026-10-12

Status: Proposed

Supersedes the `bin/` ownership in [0006](0006-runtime-substitution.md)
and the "never ships `bin/`" clause of
[0016](0016-plugin-root-code-ships-when-skills-reference-it.md). The
rest of both records stands.

## Context

[0006](0006-runtime-substitution.md) made `bin/` the directory of
interpreter symlinks the runtime pass points rewritten shebangs at,
and [0016](0016-plugin-root-code-ships-when-skills-reference-it.md)
drew the consequence: a package never carries an upstream `bin/`,
since the copy would overwrite assembly's own. Since
[0013](0013-assembly-lives-in-flox-agent.md) the pass runs inside
`agent-stacks assemble-plugin`; `buildAgentPlugin` has not written
`bin/` itself since.

Claude Code puts a plugin's `bin/` on the Bash tool's `PATH` while the
plugin is enabled, after the user's own entries, and the Agent Plugins
spec gives `./bin/server` as its example of a plugin-relative server
command. Three packages here come from repositories with a `bin/` at
the plugin root as import sees it — `agent-plugin-chisle`,
`agent-plugin-agent-browser` and `agent-plugin-agent-deck` — and each
lost what Claude's own install puts on the Bash tool's `PATH` to the
ownership rule, with nothing in `import.warnings` to say so. None of
the three files is a working command by itself: an `npx` installer, a
diagnostic bound to its author's machines, and the npm wrapper of a
native binary the repository does not hold. (`agent-plugin-netlify-skills` keeps a
`bin/` at its repository root, but import roots that plugin at
`agent-plugin/`, which has none; which root wins is a separate
question.) agent-stacks ADR 0052 records the analysis and the
alternative it rejected (relocating upstream's `bin/` into the
client-extension namespace).

## Decision

We will let an upstream `bin/` ship at the plugin root, beside the
interpreter links. `skills/` is the only plugin-root entry assembly
owns.

- `pluginRootPaths` may name `bin`. The builder copies it as any other
  entry, before the runtime pass, so its scripts are rewritten, pinned
  and guarded like a skill's. Its interpreters appear in
  `requiredRuntimes`.
- The runtime pass links each interpreter into `bin/` as before. An
  entry already there under an interpreter's name fails the build,
  naming both paths; import refuses the same collision first and
  prints the `--path-ignore` path that keeps the directory out.
- A passed-through tree's own `bin/<interpreter>` is the same
  collision, where the pass used to take it for the link.
- [0011](0011-skill-and-tool-are-separate-packages.md) stands. A skill
  and the tool it drives remain separate packages; agent-browser's
  `bin/agent-browser.js` is the tool's npm wrapper, which Claude's own
  install carries too, not the tool, and the package still does not
  carry the native binary it spawns. A `bin/` holding nothing the
  package can run can be kept out with `--path-ignore` on it, recorded
  in that package's import flags.
- Import lists the files a shipped `bin/` holds in `import.warnings`,
  as it lists hook handlers, so the hourly replay's `source.json` diff
  names each command a package adds to Claude's `PATH`.

## Consequences

- `+` The three packages above carry what Claude's own install puts
  on the Bash tool's `PATH`, under the same names.
- `+` `./bin/server`, the spec's own example, resolves in a package.
- `+` No change to `buildAgentPlugin` or to any check here: the
  behaviour lives in the binary, and its tests with it
  ([0017](0017-checks-live-where-the-behaviour-lives.md)). Packages
  change when `pkgs/agent-stacks` moves to a binary carrying
  agent-stacks ADR 0052.
- `-` A plugin whose `bin/` names an interpreter it also uses cannot
  be imported until someone chooses between its launcher and the
  pin.
- `-` A `bin/` added upstream to a Claude plugin reaches its package
  through the hourly replay, and its files land on Claude's `PATH`;
  the listing in `import.warnings` shows each by name in the diff.

---

Records AI-1004 on this side. The decision and its analysis are
agent-stacks ADR 0052.
