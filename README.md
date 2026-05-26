## Description   

Connects directly to the postgres database.  
Scans she database for images added recently   
Requests the vectors from the ML server for the keywords   
Checks to see if they match.   
Performs actions as required using the Immich API   
Adds the entry and action taken to a sqlite db to save rescan time.  



## How to use   

Copy `env.example` to `.env` and configure it.  Postgres works either way:
- **With password** (recommended for Immich-in-Docker setups, since the container's Unix socket isn't visible to this script) — set `DB_PASSWORD` in `.env`, and the script connects via TCP to `DB_HOST:DB_PORT`.
- **Without password** (peer auth, original upstream behaviour) — leave `DB_PASSWORD` commented out and run the script as the OS user that the Postgres role maps to.

Any albums you want to use must be created in Immich first — the script does not create albums, it searches for a matching name.  All keywords, albums and actions are configured in `config.json`.

## CLI flags (this fork)

```
--dry-run            Compute matches and print intended actions but do NOT call
                     Immich write APIs (album add / archive). Safe rehearsal mode
                     for tuning new keyword thresholds without risk of mass
                     mis-classification.
--rate-limit-ms N    Sleep N milliseconds between per-asset Immich-write calls
                     and between ML embedding requests. Default 100ms
                     (~10 calls/sec). Env override: RATE_LIMIT_MS.
--full-scan          Process every asset, ignoring timestamps and the
                     processed-IDs SQLite cache.
--full-unarchived-scan
                     Same as --full-scan but skip already-archived assets.
--ignore-completed   Reprocess assets even if they're in the processed-IDs cache.
```

Recommended first run: `python main.py --full-scan --dry-run` with one conservative keyword (e.g. `min_similarity: 0.30`) — review the printed `[DRY-RUN] would add ...` lines, then re-run without `--dry-run` once thresholds look right.

I have worked with a couple people and their requirements to come up with the thresholds, and keywords used for Documents, and NSFW. 
Keywords and thresholds for screenshots is not working well though.  

Running debug_asset.py on a asset id will show what distance every keyword is from said asset.  

Configure it to run using cron, or some other scheduler.  


## Disclaimer   

I am not tech support, I am simply putting this out there incase anyone else wants it.   
It has only been tested on 2 installations, one manually set up on ubuntu, and one using nixos flakes.  


## Code Origin, Author and License

This is not hand coded.  Well, a chunk of it is, but in reality, at least 3/4 was written by Gemini 2.5-pro-exp-03-25 as an agent in Cursor 0.48   
Therefor I feel anything less permissive than MIT Licence would be dishonest.   So MIT is is.

### Fork additions (SlyWombat)

- `--dry-run` flag — gates the album-add and visibility-change calls so you can rehearse new keyword thresholds.
- `--rate-limit-ms` flag — modest per-call sleep to avoid thundering-herd against ML or REST APIs (default 100ms).
- Optional `DB_PASSWORD` — script now supports TCP+password auth for Immich-in-Docker setups where the postgres Unix socket isn't reachable from the host.
- README + env.example updated to document the above.
