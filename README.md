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
  git clone --depth 1 git@github.com:gniezen/agent-devcontainer.git "$tmp"
  cp -r "$tmp/.devcontainer" .
  rm -rf "$tmp"
  echo "Copied template into ./.devcontainer"
  echo "Now run: dc up"
}
```

Run `dcinit` in your workspace folder. You can then use `dc up` and `dc exec pi` or `dc exec claude`.

To update Claude: `dc exec claude update`.

## Updating when the scripts change

Replacing the files in `.devcontainer/` (e.g. by deleting the folder and
re-running `dcinit`) is not enough on its own — the existing container was
built from the old scripts. Rebuild and recreate it with:

```
dc up --remove-existing-container
```

Your Claude login and pi config survive: they live in named volumes keyed by
the workspace, which are reattached to the new container. Anything else you
changed inside the old container's filesystem is lost, so copy that out first.

## Cleanup

The devcontainer CLI has no `down` command, so containers keep running (and
accumulating) until you stop them yourself. Helpers for .bashrc, to use
alongside `dc`:

```
# Stop the current project's container (cheap to restart with dc up)
dcstop() {
  docker ps -q --filter "label=devcontainer.local_folder=$PWD" | xargs -r docker stop
}

# Remove the current project's container entirely (config volumes survive)
dcdown() {
  docker ps -aq --filter "label=devcontainer.local_folder=$PWD" | xargs -r docker rm -f
}
```

To see what has piled up across all projects:

```
docker system df
docker ps -a --filter label=devcontainer.local_folder \
  --format 'table {{.Names}}\t{{.Status}}\t{{.Label "devcontainer.local_folder"}}'
```

Safe ways to reclaim space:

```
docker image prune     # dangling layers left over from rebuilds
docker builder prune   # build cache (regenerated on next rebuild)
```

Avoid `docker volume prune` and be careful with `docker system prune`: the
`claude-config-*` volumes hold your Claude logins, and `system prune` also
deletes every stopped container, including devcontainers you meant to keep.
Removing a single project's container with `dcdown` is always safe — its
volumes persist and reattach on the next `dc up`.

This repository includes init-firewall.sh adapted from
https://github.com/anthropics/claude-code (MIT licensed).
Original copyright: Anthropic, PBC.
