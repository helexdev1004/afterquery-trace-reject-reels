# Trace rejected sheets back to their paper reels

A harbor-format benchmark task, authored for the AfterQuery Experts programme as data
annotation for AI evaluation. Task slug `afterquery/trace-reject-reels` — **Operations / Carton Printing**.

An AI agent gets a container, this brief and nothing else. It works alone for
4 hours. Then a sealed verifier it never sees scores the result `0` or `1`.

## What it asks for

Which paper reel each rejected sheet came from

The full brief is in [`instruction.md`](instruction.md).

| | |
| --- | --- |
| **Deliverable** | `/app/reject_reels.csv` |
| **Agent budget** | 4 hours (14400s) |
| **Expert estimate** | 4 hours |
| **Machine** | 1 CPU, 2048 MB RAM, 0 GPU |
| **Verifier budget** | 300s |
| **Tags** | `print-production`, `traceability`, `quality-claims`, `data-cleaning`, `python` |

## Why it is hard

A carton plant's quality technician or production planner does this whenever a claim is raised against a board mill: rejected sheets have to be tied to the reel they were cut from, and the only link between the die-cutter's count and the press's count is how the pallet travelled between the two machines. The data here is synthetic and was made truth first - 54763 sheets were printed reel after reel onto 6 pallets, each sheet keeping its reel and its caliper, and only then was each pallet fed through the die-cutter and the die-cutter's record written. The 42 sealed lines are read from the press's side and are exact; nothing is computed at marking time.

<details>
<summary>Full note</summary>

Three of the six pallets are not recorded as the pressroom's note describes, and nobody could miss it: their calipers are in thousandths of an inch, in millimetres and in tenths of a micrometre. Each is a setting and nothing more. Left alone, each spoils a pallet's seven lines, and only 4 lines may be out; with the note believed on every point the score is 14 of 42. The deciding fact is that 2 pallets never went through the pallet turner. The note says every pallet is turned, so that the first-printed sheet is fed first; these two were fed last-printed first, and a reject's count on the die-cutter is its count from the other end of the press's stack. No record says so, and the note's reading fails silently - every count still lands on some reel - at a cost of 14 lines; one of the two pallets also has an odd gauge setting and the other is recorded exactly as the note says. The level of the caliper is no guide: all the board is one grade, the reading wanders by several micrometres with the room over a job, and the reel-by-reel means on a pallet differ by up to 12.0 um for that reason alone, in no relation to the reels. A reel does differ from the next in how even it is from sheet to sheet. Taking each sheet's distance from the mean of its neighbours, and comparing the middle value over the 400 sheets either side of each reel change the press booked, the evenness steps by at least 2.02 times as much with a pallet read the right way round as the wrong way. Feeder hiccups throw short bursts of wild readings across every reel, so a table of spread by reel is blurred in either orientation; and because a reel runs on from one pallet to the next, both ends of a pallet look alike. tests/floor_check.py measures all of this against the sealed lines and refuses to pass otherwise: of 16 readings the record's own gets 42 and the best other 35, against a bar of 38.

</details>

## How it is verified

The verifier reads /app/reject_reels.csv with the csv module and compares each reject's reel, as a string, and caliper, within 2 micrometres, with /tests/sealed.json; at most 4 of the 42 lines may be out. Nothing the agent wrote is imported or executed. The sealed lines exist only in the verifier image, where /tests is mode 700, and the agent's container never mounts it.

<details>
<summary>Full note</summary>

Two micrometres covers the rounding of a reading converted from thousandths of an inch and nothing else. On the two pallets that were not turned, the rejects were drawn only from sheets whose count from the other end falls on a different reel, so none is right by chance, and no reject lies within 300 sheets of either end of a pallet. No reel is attested anywhere in the agent's container and nothing has to balance: a claim made on the note's reading is as self-consistent as the right one. The reward is binary and is written to /logs/verifier/reward.txt on every path; an empty container scores 0 and the reference scores 1. The generator is seeded and shipped.

</details>

<details>
<summary>Reference approach (spoiler)</summary>

solution/claim.py reads only /app and writes /app/reject_reels.csv with the standard library. It parses the press ticket into pallets and booked reel changes, groups the die-cutter's record by pallet, and tells each pallet's gauge setting from the middle of its readings. For each pallet it computes every sheet's distance from the mean of the sheets fed before and after it, and scores the booked reel changes twice - once as if the pallet was turned, once as if it was not - by how far the middle of that distance moves across each change; the better score says which way the pallet was fed. A reject's count is then carried to the press's count, the ticket gives its reel, and the record gives its caliper. In the task's own container it places 42 of 42.

</details>

## Layout

```
instruction.md          the brief - the only thing the agent is told
task.toml               artifacts, timeouts, resources, metadata
environment/            Dockerfile and data for the agent's container
solution/solve.sh       reference solution; scores 1 by construction
tests/                  verifier image, grading logic and ground truth
```

## Running it

With [harbor](https://github.com/laude-institute/harbor) installed and Docker running:

```bash
uv tool install harbor

harbor run -p . -a oracle -e docker   # reference solution - must score 1
harbor run -p . -a nop    -e docker   # nothing happens    - must score 0
```

Those two runs are the calibration the bundle has to satisfy: the oracle run substitutes
`solution/solve.sh` for the agent and must score `1`, the nop run does nothing and must
score `0`. Anything in between means the verifier is measuring the wrong thing.

## Reading this

`tests/` and `solution/` hold the answers. To try the task honestly, read only
`instruction.md` and `environment/`.

---

Part of a set of 38 tasks indexed at
[afterquery-data-annotation](https://github.com/helexdev1004/afterquery-data-annotation).
