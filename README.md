Just my own devcontainer scripts for running local LLMs and Claude Code.

First, do a `npm install -g @devcontainers/cli`.

Use with the following in .bashrc:

```
dc() {
  devcontainer "$1" --workspace-folder . "${@:2}"
}

dcinit() {
  if [ -d ".devcontainer" ]; then
    echo "Already has .devcontainer/"
    return 1
  fi
  local tmp=$(mktemp -d)
  git clone --depth 1 git@github.com:gerritniezen/agent-devcontainer.git "$tmp"
  cp -r "$tmp/.devcontainer" .
  rm -rf "$tmp"
  echo "Copied template into ./.devcontainer"
  echo "Now run: dc up"
}
```

Run `dcinit` in your workspace folder. You can then use `dc up` and `dc exec pi` or `dc exec claude`.

This repository includes init-firewall.sh adapted from
https://github.com/anthropics/claude-code (MIT licensed).
Original copyright: Anthropic, PBC.
