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

**whatsthis** è un automator di ricognizione basato su `nmap`: un singolo script Bash
che esegue un workflow di scansione completo, affidabile e pensato per il workflow
da CTF / OSCP. L'obiettivo è semplice: lanci un comando, e ottieni porte, servizi,
versioni e i **comandi di enumerazione successivi** già pronti da copiare — senza
perdere porte per strada.

## Caratteristiche

- **Workflow multi-fase**
  - *Fase 0* — quick-win top 1000 per un foothold immediato
  - *Fase 1* — full TCP SYN scan su tutte le 65535 porte
  - *Fase 2* — version detection + NSE default (`-sCV`) **solo** sulle porte aperte
  - *Fase 3* — UDP top 100 (`-sU -sV`)
- **Due modalità**
  - *sequenziale* (default) — un solo stream, timing conservativo: massima affidabilità
  - *parallela* (`-P`) — porte divise in blocchi concorrenti + UDP in parallelo al TCP;
    numero di job auto-rilevato dai core (cap 8) o forzato con `-j N`
- **Garanzie anti-perdita di porte**
  - `-Pn`: un host che blocca il ping non viene saltato
  - ogni scansione è valida solo se `nmap` segnala `Nmap done`, altrimenti viene **ritentata**
  - niente merge silenzioso di dati parziali: i range non confermati sono elencati
    e lo script esce con codice ≠ 0
- **Risoluzione hostname** — reverse DNS + mappatura opzionale in `/etc/hosts`
- **Suggerimenti per servizio** — `next-steps.txt` con i comandi di enumerazione pronti
  (web, smb, ftp, ssh, ldap, kerberos, mssql/mysql/postgres, rdp, winrm, ...) — *solo
  enumerazione, OSCP-safe*
- **Report Markdown** — `summary.md` riutilizzabile nel report
- **Multitarget** (`-f targets.txt`) — una cartella per host
- **Resume/skip** (`-r`) — salta le scansioni già completate
- **Verbose** (`-v`) — stampa i comandi lanciati

## Requisiti

- `nmap`
- `bash`
- privilegi `sudo` (necessari per `-sS` / `-sU`)
- *(opzionali)* `dig`/`host`/`nslookup` per il reverse DNS; gli strumenti citati nei
  suggerimenti (`feroxbuster`, `enum4linux-ng`, SecLists, ...) per i passi successivi

## Installazione

```bash
git clone https://github.com/Givaa/whatsthis.git
cd whatsthis
chmod +x whatsthis
```

## Uso

```
./whatsthis [-v] [-P|-S] [-j N] [-r] <IP> <nome>
./whatsthis [opzioni] -f targets.txt
```

### Opzioni

| Opzione | Descrizione |
|--------|-------------|
| `-P` / `-S` | modalità parallela / sequenziale (default: sequenziale) |
| `-j N` | numero di job in parallelo (implica `-P`; `0` = auto) |
| `-r` | resume: salta le scansioni già completate (`Nmap done`) |
| `-f FILE` | multitarget: una riga per host, `IP [nome]` (`#` = commento) |
| `-v` | verbose: stampa i comandi lanciati |
| `-h` | help |

### Esempi

```bash
# Singolo target, sequenziale (massima affidabilità)
./whatsthis 10.10.10.5 box01

# Parallelo con job auto-tuned
./whatsthis -P 10.10.10.5 box01

# Multitarget, parallelo, riprende da dove era rimasto
./whatsthis -P -r -f targets.txt
```

Esempio di `targets.txt`:

```
# lab targets
10.10.10.5 box01
10.10.10.6 dc01
10.10.10.7
```

## Output

Tutto finisce in una cartella per host (`./<nome>/`):

| File | Contenuto |
|------|-----------|
| `quick.txt` | quick-win top 1000 (foothold immediato) |
| `allports.txt` | full TCP |
| `deep.txt` | version detection + NSE |
| `udp.txt` | UDP top 100 |
| `next-steps.txt` | comandi di enumerazione suggeriti per servizio |
| `summary.md` | report Markdown |

## Nota OSCP

whatsthis automatizza solo **ricognizione ed enumerazione** e si limita a *suggerire*
i comandi successivi: non esegue exploitation automatica, in linea con le regole d'esame.
Verifica sempre i percorsi degli strumenti (es. SecLists) nei comandi suggeriti rispetto
alla tua macchina.

## Autore

Giva — [@Givaa](https://github.com/Givaa)
