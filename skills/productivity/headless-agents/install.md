# Installing Nightshift

Nightshift needs Node 22 or later, Claude Code's login, and a shell for the gates (`/bin/sh`). It
runs on macOS and Linux directly. On Windows it runs inside WSL or Docker Desktop, because gates are
shell commands.

## The login token (every machine)

On a computer with a browser, run once and keep the printed token:

```sh
claude setup-token
```

It goes in a secrets file only the user can read. Keep the file outside any project folder:

```sh
mkdir -p ~/.nightshift && touch ~/.nightshift/secrets.env && chmod 600 ~/.nightshift/secrets.env
```

The user pastes `CLAUDE_CODE_OAUTH_TOKEN=<token>` into it themselves; never type, print or echo the
token. Other keys an agent needs (`RESEND_API_KEY`, `REPORT_TO`...) go in the same file.

## This computer (macOS or Linux; Windows inside WSL)

```sh
git clone https://github.com/ennio-datatide/nightshift.git ~/.nightshift/src
cd ~/.nightshift/src && npm ci && npm run build && npm link
mkdir -p ~/agents ~/.nightshift/state
cp examples/hello.md ~/agents/
nightshift run hello --agents ~/agents --state ~/.nightshift/state --secrets ~/.nightshift/secrets.env
ln -sfn ~/.nightshift/src/skills/nightshift-agents ~/.claude/skills/nightshift-agents
```

The last line gives Claude Code Nightshift's own skill, `nightshift-agents`, which writes the agents.

Then keep the daemon running with the computer: on a Mac, the LaunchAgent in
`~/.nightshift/src/skills/nightshift-agents/triggers.md` ("Keeping agents alive"); on Windows, a Task Scheduler task "At log on"
that starts `wsl nightshift daemon ...`; on Linux, a systemd user service with `Restart=always`.

Put `~/agents` in git: the folder of agent files is the whole configuration.

## A VPS (runs while the computer is off)

Install on this computer first (above): the agents are written and tested here, with the
`nightshift-agents` skill, and then copied up. Any small Linux server with Docker. On the server:

```sh
git clone https://github.com/ennio-datatide/nightshift.git ~/nightshift && cd ~/nightshift
docker build -t nightshift .
cp compose.example.yaml compose.yaml
mkdir -p agents workspaces && cp examples/hello.md agents/
touch secrets.env && chmod 600 secrets.env   # the user pastes the token in
docker compose up -d
docker compose exec nightshift nightshift run hello --agents /agents --state /state --secrets /run/secrets.env
```

The README's "Docker" section explains each mount. Set `user:` in `compose.yaml` to the server
user's `uid:gid` when agents write in folders other tools share, and `TZ` for agents without a
`timezone`. Keep the agent files in git on the user's computer and copy them up with `rsync` or
`git pull` on the server; the daemon picks up changed files on its own.

## Checking it is alive

`nightshift list` shows every agent, its next run and its last result. `nightshift runs` shows the
history. On a VPS, prefix both with `docker compose exec nightshift`.
