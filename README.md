# opentab-log

Daily checkpoints for one public record stream, so that "it was written on this day"
is a fact somebody else can check, not a claim this domain makes about itself.

## What is in here

One JSON file per day: `checkpoints/YYYY-MM-DD.json`.

```json
{
  "spec": "opentab/checkpoint/v1",
  "recorder": "walnutlink.ai",
  "stream": "walnutlink-main",
  "day": "2026-10-26",
  "seq_from": 55, "seq_to": 78,
  "count": 24,              // records added that day
  "total_count": 78,        // records in the stream as of that day
  "root_hash": "sha256:...",// Merkle root over that day's records
  "prev_checkpoint_hash": "sha256:...",
  "checkpoint_hash": "sha256:...",
  "backfilled": false,
  "anchors": [ ... ]
}
```

## How to check it yourself

Nothing here asks you to trust walnulink.ai. Three things you can verify with a short script:

1. **The day's root.** Fetch the day's records, canonicalise each as
   `json.dumps(rec, sort_keys=True, ensure_ascii=False, separators=(",", ":"))`, build the
   Merkle tree the RFC 6962 way (`SHA256(0x00 || leaf)`, `SHA256(0x01 || L || R)`,
   left subtree is the largest power of two below `n`), and compare to `root_hash`.
2. **The chain.** `prev_checkpoint_hash` of day N must equal `checkpoint_hash` of day N-1.
3. **No missing records.** `total_count` must equal the real number of records. A hash chain
   cannot catch a stream that is quietly shortened — a count can.

`checkpoint_hash` is the SHA-256 of the checkpoint body **excluding** `anchors`,
`built_at` and `checkpoint_hash` itself. Those three are not part of the commitment.

## What the git commit adds

A commit date is not something the recorder can edit afterwards. So even if the recorder
rewrote its whole history and produced a perfectly consistent chain, it could not move
these commits back in time.

## What this does NOT prove

- **Not that the records are true.** A checkpoint commits to what was written, nothing more.
- **Not that nothing was hidden.** If records were never written, no chain shows them.
- **Not that the author is honest about confidence.** Only the sequence is anchored.

## A note on the first five days

`2026-10-22` through `2026-10-26` carry `"backfilled": true`. Their timestamps come from
the recorder's own database and were anchored after the fact, not on the day.
The honest reading: the chain starts here, but the first five days have no external proof
of when they were written. Everything from `2026-10-27` onward is anchored on the day.

## Why the early days are kept

Because a record that only starts once it looks good is not a record.
