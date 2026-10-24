# dockman

An interactive terminal manager for the Docker containers on the machine it runs on.
It is a single Python 3 file that uses only the standard library. It works through the
ordinary `docker` CLI, so you don't need a management container or agent.

## Install

```sh
sudo install -m 755 dockman /usr/local/bin/dockman
dockman
```

Requirements:
- Python 3.8 or later
- the `docker` CLI
- `docker compose` (v2) or `docker-compose` for compose features
- a user that can run docker without sudo (a member of the `docker` group)
- `less` and an editor such as `$EDITOR`, `nano` or `vim` (optional, but recommended)

## The main screen

The main screen lists every container, sorted by compose project and name. Each row shows:
- status and uptime
- live CPU and memory use
- restart policy
- compose project
- image
- published ports

Rows are coloured:
- **Green:** running.
- **Yellow:** paused, starting or busy.
- **Red:** unhealthy, restarting or failed.
- **Dim:** stopped cleanly.

The list refreshes every few seconds. Press **Enter** on a container to open its action menu,
or use these keys directly:

| Key | Action |
|---|---|
| `s` / `x` / `r` | start (or unpause) / stop / restart |
| `X` | **stop and disable auto-restart**: sets the restart policy to `no`, then stops |
| `p` / `K` / `d` | pause and unpause / kill / remove (optionally with its anonymous volumes) |
| `l` / `L` / `f` | logs (last 5000 lines in `less`) / logs filtered by since, tail or search / follow live |
| `e` / `!` | shell inside the container / run a command in it |
| `i` / `I` / `t` / `S` | info summary / raw inspect / processes / live stats |
| `u` / `n` | change restart policy / rename |
| `E` | export the container as an equivalent `docker run` command and `compose.yaml` |
| `U` | pull the latest image and recreate the container (compose containers only) |
| `c` | actions for the container's compose project |
| `a` | add: guided `docker run`, paste a `docker run` command, new compose project, or start an existing compose file |
| `P` | compose projects: up, pull and update, start, stop, restart, down, logs, ps, config, edit the file |
| `D` | disk usage and pruning |
| `m` | images, volumes, networks, Swarm services, events, disk usage, prune, docker info |
| `/` / `A` / `R` | filter / toggle all or running-only containers / refresh |
| `?` / `q` | help / quit |

## Restart policies and stopping

A plain stop (`x`) is never undone by the restart policy while the Docker daemon keeps
running. However, a container with `--restart=always` does start again when the daemon
or the server restarts.

`X` changes the policy to `no` and then stops the container, so it stays down whatever
happens. Turn the policy back on later with `u`. Compose containers get their own policy
back the next time their project is brought `up`.

## Swarm services

Containers run by a Docker Swarm service (for example from `docker stack deploy`) show
`swarm` in the RESTART column and `stack:<name>` as their project. Swarm replaces any such
container that stops, whatever its restart policy, so stopping the container alone never
sticks. For these containers, `s`, `x`, `X`, `r`, `K` and `d` open a menu for the service
instead:
- stop it by scaling to 0 replicas (it stays stopped), or start it by scaling up again;
- restart all its tasks (rolling);
- view or follow the service logs, list its tasks, or inspect it;
- remove the service, or the whole stack.

`m` → Swarm services lists every service. These actions need a Swarm manager node.

## Adding containers

- **Guided run:** asks for the image, name, ports, volumes, environment, restart policy,
  network, extra options and command. It shows the generated `docker run` command before
  running it, and can save the result as a compose project instead.
- **Paste a `docker run` command:** runs it as given, and offers to add `-d` if needed.
- **New compose project:** write the `compose.yaml` in your editor from a template, or
  paste it. It is saved to `~/docker/<project>/` by default; set
  `DOCKMAN_COMPOSE_ROOT` to use a different location. The file is validated with
  `docker compose config` and then started.
- **Existing compose file or directory:** runs `docker compose up -d` on it.

## Disk usage and pruning

`D` shows a summary of Docker's disk usage (images, containers, volumes and build cache,
with how much is reclaimable) and the free space on Docker's filesystem. From there you can:
- see a per-item breakdown;
- prune images older than a chosen age (for example `30d`), either all unused images or
  only untagged ones. Age is measured from the pull date by default (when the image was
  pulled or tagged on this machine), or optionally from the build date (the CREATED column
  in `docker images`). You see the list before anything is removed. Images used by any
  container, even a stopped one, are always kept;
- prune stopped containers, dangling or all unused images, volumes, networks, build cache,
  or run a whole-system prune.

Every destructive action asks for confirmation first.

## Licence

MIT. See [LICENSE](LICENSE).
