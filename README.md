# computational-life-results

Run archives of BFF primordial soup experiments, produced by
[computational-life](https://github.com/peterseb1969/computational-life) (`bff_archive.py`).
Published at https://github.com/peterseb1969/computational-life-results.
One JSON file per run in `archive/`, named `<host>-<seed>.json`. Each archive is
self-contained: parameters, event epochs, the winning species and families with
raw bytes, the ancestry of the winners with replayed birth tapes, and the metrics log.

## Adding a run

On any machine with the code:

```bash
git clone https://github.com/peterseb1969/computational-life-results.git   # once
export BFF_ARCHIVE_DIR=/path/to/computational-life-results/archive            # or --archive-dir
python3 bff_soup.py --num 131072 --epochs 60000 --seed 47 --max-steps 8192 --metric-sample 32768 \
        --stop-selfreps 65536 --stop-after 2048
# the archive is written when the run ends or is stopped; for a finished run: python3 bff_archive.py runs/47
cd /path/to/computational-life-results && git add archive && git commit -m "Add <host> run 47" && git push
```

## Protocols

Runs are grouped for statistics by their **protocol**, derived from the parameters
unless given with `--protocol`:

| Protocol | Programs | Step budget | Mutation |
|----------|----------|-------------|----------|
| `128k-8192` | 131072 | 8192 | none |
| `128k-8192-mut4096` | 131072 | 8192 | one byte in 4096 per epoch, 0.000244 (the paper's default) |
| `128k-32768` | 131072 | 32768 | none (the original fork's default) |
| `128k-8192-heads` | 131072 | 8192 | none; the first two bytes of a tape set the heads (`--heads`) |

A suffix `-init-<name>` marks a run whose initial soup was not uniform random bytes (`--init-dist`): `ops50` half instructions, `ops100` instructions only, `winners` instructions at the frequencies of the collection's winners plus stop and alignment bytes, `custom` for an explicit key:weight list (recorded in the archive).

## Reading the collection

```bash
python3 bff_compare.py /path/to/computational-life-results/archive             # table of runs
python3 bff_compare.py /path/to/computational-life-results/archive --survival  # fraction transitioned by epoch, per protocol
python3 bff_compare.py /path/to/computational-life-results/archive --families  # winners across runs
python3 bff_compare.py /path/to/computational-life-results/archive --csv runs.csv
```

A run that ended without a transition counts as censored: the survival table uses
the Kaplan-Meier estimate, so it contributes for the epochs it covered.

## Experiments

`experiments/origin-rate/` holds the removal records of origin-rate runs (`--cull-replicators`): every
self-replicator is removed as soon as it is seen, so the file lists when and what arose, not what won.
`bff_query.py culls` reads a run directory; the JSON here is the same list with the run's parameters.
These runs have no archive in `archive/` and do not enter the survival statistics.

## Duplicates

`duplicates/` holds archives of runs that repeat a seed already in `archive/` on another
machine. Same seed means the same universe (the pairing of every epoch derives from the
seed), so they are cross-machine determinism checks, not independent observations, and
`bff_compare.py` does not read them. `mac-mini-44` reproduced `ps-macbook-pro-44` exactly:
same events, same winners with identical counts.

## Reproducing a run

Every run is deterministic: the seed fixes the initial soup and the pairing of every epoch.
To regenerate a run directory (needed for the viewer's lineage and stepper tabs), run the
simulator with the archive's `seed_label` and protocol, for example for `ps-macbook-pro-macbook-1`:

```bash
python3 bff_soup.py --seed macbook-1 --num 131072 --max-steps 8192 --metric-sample 32768 --epochs 60000
```

About 20 minutes on an M4 Pro. The result is byte-identical to the original run.
