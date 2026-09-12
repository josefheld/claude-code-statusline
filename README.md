# Claude Code Statusline

> **Moved.** This statusline now lives in the [claude-plugins](https://github.com/josefheld/claude-plugins) marketplace as the `statusline` plugin. This repo is archived and no longer maintained, the script below is the last standalone version.
>
> ```
> /plugin marketplace add josefheld/claude-plugins
> /plugin install statusline@josefheld
> ```
>
> Then let the skill set it up (`"set up my statusline"`), or run the installer once:
>
> ```sh
> sh ~/.claude/plugins/marketplaces/josefheld/plugins/statusline/scripts/install-statusline.sh
> ```

A custom statusline script for [Claude Code](https://claude.ai/claude-code) that displays real-time session information in your terminal: model, context window, cost, both rate limits, git state, session runtime.

![Statusline preview](screenshot.png)

```
🤖 Opus 5 T | 🧠 ██████░░ 75% | 💰 $0.42 | ⏱️ 5h ████░░░░ 45% ↻14:30 | 📁 my-project | 🌿 main +3 ~5 | 🕐 01:23:45
```

## Why it moved

A statusline is not a plugin component, so this could never be a plugin in the usual sense: Claude Code reads it from `statusLine` in `settings.json`, and something has to write that entry. The plugin ships an installer that does it.

What the move buys, and what this repo could not offer:

- **Updates arrive.** `settings.json` points at the script inside the plugin directory, so `claude plugin update` also updates the script. The `cp ... ~/.claude/statusline-command.sh` in the old setup below goes stale the moment anything changes here.
- **Segments are configuration.** Which segments appear, and in which order, is `CC_STATUSLINE_SEGMENTS=model,context,cost,git` in front of the command, not a code edit. Plus bar width, the yellow and red thresholds, and the reset time format.
- **Three bugs are fixed** that are still present in the script in this repo:
  - `printf` and `awk` parse `62.4` through `LC_NUMERIC`. In any locale with a decimal comma (`de_AT`, `fr_FR`, ...) that is an `invalid number`, so the context bar reads `0%` in red and the cost prints `$0,42`. Fixed by exporting `LC_NUMERIC=C` while leaving `LC_CTYPE` alone, so the bar characters still render.
  - A payload without `.rate_limits` printed a literal `null%`.
  - `%p` is an empty string in most non-English locales, turning `2:30PM` into a bare `2:30`. The reset time is 24h by default now.
- **A self-check.** `test-statusline.sh` asserts the segment behaviour instead of leaving it to the next session to notice.

## The old standalone setup

Still works if you cloned this repo, and gets no further fixes.

```sh
cp statusline-command.sh ~/.claude/statusline-command.sh
chmod +x ~/.claude/statusline-command.sh
```

```json
{
  "statusLine": {
    "type": "command",
    "command": "sh ~/.claude/statusline-command.sh"
  }
}
```

Requires [`jq`](https://jqlang.github.io/jq/) and `git`.

## Credits

Original script by [@danielmackay](https://github.com/danielmackay), see [danielmackay/claude-code-statusline](https://github.com/danielmackay/claude-code-statusline). Enhanced here with a single-parse `jq` call, one-line POSIX `sh` output, thinking mode, effort level, both rate limits with reset times, worktree, active agent and session name.
