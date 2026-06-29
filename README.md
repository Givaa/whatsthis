# whatsthis

```
                       .,,uod8B8bou,,.
              ..,uod8BBBBBBBBBBBBBBBBRPFT?l!i:.
         ,=m8BBBBBBBBBBBBBBBRPFT?!||||||||||||||
         !...:!TVBBBRPFT||||||||||!!^^""'   ||||      __      ___         _      _____ _    _    ___
         !.........||||                     ||||      \ \    / / |_  __ _| |_ __|_   _| |_ (_)__|__ \
         !.........||||  $ nmap_            ||||       \ \/\/ /| ' \/ _` |  _(_-< | | | ' \| (_-< /_/
         !.........||||                     ||||        \_/\_/ |_||_\__,_|\__/__/ |_| |_||_|_/__/(_)
         !.........||||                     ||||
         !.........||||                     ||||      nmap automator
         !.........||||                     ||||      by Giva · github.com/Givaa
```

**whatsthis** is a single Bash script that automates nmap recon: run one command and
get ports, services, versions, and ready-to-paste next enumeration commands — without
missing ports. Built for CTF / OSCP workflows.

## Features

- **Phases**: quick (top 1000) → full TCP (`-p- -sS`) + version+NSE (`-sCV`, open ports only) → UDP top 100 (`-sUVC`)
- The quick scan runs **without `-Pn`** as a liveness check: if the host looks down it says so and asks whether to continue (auto-continues in background)
- **UDP runs in the background, concurrent with the full TCP scan** (one extra process, cleaned up on Ctrl-C)
- nmap output and the exact command being run are shown on screen
- **Anti-loss**: `-Pn` (don't skip ping-blocking hosts); each scan must reach `Nmap done` or it's retried; unconfirmed ranges are listed and the script exits non-zero
- **Hostnames**: discovered from services (smb / ssl-cert / http redirect) and printed. With `-H`, whatsthis also edits `/etc/hosts` (missing aliases, idempotent) and renames the output dir to `<name>-<discovered>`. Off by default so scans never block on a prompt; `-H` asks first, add `-g` to skip the prompt
- **Per-service hints**: `next-steps.txt` with enumeration commands per service — enumeration only (OSCP-safe)
- **Markdown report**: `summary.md` with open ports and the exact commands run
- **Multi-target** (`-f targets.txt`), **resume** (`-r`); per-target and total elapsed time printed at the end
- One sudo prompt up front (kept warm) so long runs can be left unattended

## Requirements

- `nmap`, `bash`, and `sudo` (for `-sS` / `-sU`)
- optional: `dig`/`host`/`nslookup` (reverse DNS); the tools referenced in the hints (`feroxbuster`, `enum4linux-ng`, SecLists, …)

## Install

```bash
git clone https://github.com/Givaa/whatsthis.git
cd whatsthis
chmod +x whatsthis
```

## Usage

```
./whatsthis [-v] [-r] [-H] [-g] <IP> [name]
./whatsthis [opts] -f targets.txt
```

| Option | Meaning |
|--------|---------|
| `-r` | resume: skip scans already done |
| `-H` | enable /etc/hosts edits + dir rename (off by default) |
| `-g` | grant: do every op without asking (use with `-H`) |
| `-f FILE` | multi-target: one `IP [name]` per line (`#` = comment) |
| `-v` | verbose: print helper commands too |

The `name` is optional — with just an IP the output dir is the IP and the real name is
discovered during the scan.

### Examples

```bash
./whatsthis 10.10.10.5                  # IP only
./whatsthis 10.10.10.5 box01            # named
./whatsthis -H 10.10.10.5 box01         # also map /etc/hosts (asks first)
./whatsthis -r -f targets.txt           # multi-target, resume
```

## Output

Everything goes into `./<name>/`:

| File | Contents |
|------|----------|
| `quick.txt` | quick-win top 1000 |
| `allports.txt` | full TCP |
| `deep.txt` | version + NSE |
| `udp.txt` | UDP top 100 |
| `next-steps.txt` | per-service enumeration hints |
| `summary.md` | Markdown report (ports + commands run) |

## OSCP note

whatsthis only automates **recon/enumeration** and *suggests* the next commands — no
automatic exploitation, in line with exam rules. Check the tool paths (e.g. SecLists) in
the suggested commands against your machine.

## Author

Giva — [@Givaa](https://github.com/Givaa)
