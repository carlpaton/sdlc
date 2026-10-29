# GitHub CLI prerequisite

The GitHub CLI (`gh`), authenticated, is required for issue and PR operations. Check before doing anything else.

1. Run `command -v gh`. If it is missing, tell the user and ask whether you may install it. Do not install without a yes. If they agree, install it using the platform's package manager (for example `sudo apt install gh` on Debian/Ubuntu, `brew install gh` on macOS). Installing may need a sudo password you cannot enter, in which case ask the user to run the command themselves with `! <command>`. If you cannot install it, point them to https://cli.github.com and stop.
2. Run `gh auth status`. If it is not authenticated, ask the user to run `! gh auth login` in the prompt, since it is interactive, then re-check.
3. Continue only once both pass.
