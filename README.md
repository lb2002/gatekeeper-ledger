# gatekeeper-ledger
Public timestamps of Gatekeeper detections (salted hashes, revealed after confirmation)

## How this ledger works

[Gatekeeper](https://github.com/lb2002/gatekeeper) watches every new npm release. When it flags a package, it commits a
line `<flag time> <sha256>` to `commitments/<day>.txt` within minutes. The hash is

    sha256(salt || {"flagged_at": ..., "package": ..., "verdict": "flagged", "version": ...})

with a random 16-byte salt per flag (canonical JSON, sorted keys, no spaces). The package name stays hidden, so nobody is named
before a detection is verified.

Once a detection is independently confirmed (OSV / OpenSSF malicious-packages, GitHub advisories, or an npm takedown), its salt
and line are published in `revealed/<month>.jsonl`. To verify a claim "Gatekeeper flagged X at time T":

1. Recompute `sha256(bytes.fromhex(salt) + line.encode())` and find the digest in `commitments/`.
2. Check the git commit time of that line: it must be earlier than the public report.

Flags that are never confirmed are never revealed.

## Update (2026-10-28 11:30 UTC): independent timestamps

Git commit dates can be set by whoever commits, so they prove nothing on their own. Since 2026-10-28 ~11:30 UTC every batch
is written as a new file `commitments/<day>/<HHMMSS>-<id>.txt` together with `<same name>.txt.ots`, an
[OpenTimestamps](https://opentimestamps.org) proof that anchors the batch in the Bitcoin blockchain through public calendar
servers. To check when a batch existed, independently of Gatekeeper and of GitHub:

    pip install opentimestamps-client
    ots upgrade <batch>.txt.ots      # once the Bitcoin attestation is available (a few hours)
    ots verify <batch>.txt.ots

Entries in the older per-day files `commitments/<day>.txt` (before 11:30 UTC on 2026-10-28) only have git dates.
