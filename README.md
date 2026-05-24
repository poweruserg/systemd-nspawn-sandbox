# systemd-nspawn-sandbox

Persistent sandboxes layered over your host filesystem using `systemd-nspawn` and kernel `overlayfs`. Install packages, run experiments, try unknown tools — all changes go into one directory you can inspect, reset, or wipe.

## How it works

`overlayfs` mounts your host `/` (read-only lower) under a writable upper layer at `<sandbox-path>/upper`. `systemd-nspawn --boot` runs on the merged view, so inside you see the full host system with all your tools and configs — but every write lands in `upper` and is trivially reversible.

## Requirements

- Linux with `overlay` kernel module
- `systemd-nspawn`, `nsenter`, `util-linux` (`script`), `sudo`

## Install

```sh
git clone https://github.com/<you>/systemd-nspawn-sandbox.git
cd systemd-nspawn-sandbox
sudo install -m 0755 sb-* /usr/local/bin/
```

## Usage

```sh
sb-create <path>            # create new sandbox + start
sb-resume <path>            # start an existing (stopped) sandbox
sb-enter  <path>            # enter as your user (green prompt)
sb-enter  <path> root       # enter as root (red prompt)
sb-list                     # show running sandboxes
sb-stop   <path>            # shut down, keep state
sb-reset  <path>            # shut down, wipe upper layer
sb-wipe   <path>            # shut down, delete sandbox dir
```

State persists between sessions and reboots — `sb-resume` to bring it back.

### Example

```sh
sb-create ~/dev
sb-enter  ~/dev
# [sb:dev] poweruser@dev:~$ sudo pacman -S htop ripgrep
# [sb:dev] poweruser@dev:~$ exit

sb-enter ~/dev root
# [sb:dev] root@dev:/# whoami
# root
# [sb:dev] root@dev:/# exit

sb-stop   ~/dev    # later: sb-resume ~/dev
sb-wipe   ~/dev    # done with it
```

Path can be absolute or relative; multiple sandboxes work fine, just use unique basenames.

Inside, confirm you're sandboxed:

```sh
systemd-detect-virt   # systemd-nspawn
echo $SB_NAME         # dev
```

## Caveats

- Submounts on separate filesystems (e.g. a separate `/home` partition) are not visible inside. Add `--bind=…` to `sb-create` if needed.
- Sandboxes share the host network namespace and kernel. Capabilities for `modprobe`, time, and `sysctl` are dropped by default.
- `pacman -Syu` works but wastes space — prefer `pacman -S <pkg>`.
- Same basename in two paths can't run simultaneously (collides on `--machine=`).

## License

MIT
