# qrpack — 3-day sprint plan

Six people, 72 hours, a working package and an even contribution graph.

Honest framing: a three-day history looks like three days. That is fine. What is
judged is whether **every member contributed real, reviewed work**, and that is
entirely achievable in three days. Do not backdate anything.

Target: ~70 commits, 18 merged pull requests, 6 contributors with comparable
columns in Insights → Contributors.

## The team

| # | Member | GitHub | Owns |
|---|---|---|---|
| 1 | Arsen Toktarbek | `arsentis` | `core.py` archiving (pack/unpack, compression) |
| 2 | Aubakir Arman | `Rustlingrass` | `core.py` framing (Header, chunk, sha256) |
| 3 | alikaiyr | `AliKa7` | `sheet.py` layout and output |
| 4 | Yerassyl | `yerassy1dev` | `scan.py` decoding and recovery |
| 5 | Arman Kali | `dreamboy17` | `cli.py`, CI |
| 6 | Aslanbek | `Hetag-byte` | `calibrate.py`, README |

Everyone has 3 issues: one per day, assigned on GitHub.

Review ring — you review the person below you, and nobody reviews their own work:

    Arsen -> Aubakir -> alikaiyr -> Yerassyl -> Arman -> Aslanbek -> Arsen

Aim for **4 commits per issue** (failing test → minimal pass → edge case →
docs). See CONTRIBUTING.md § "Commit FOUR times per feature".
Six people × 4 commits × 3 days ≈ 70 commits.


## Hour 0 — before anyone writes code (20 minutes, group chat is fine)

This does not need a call, but the result must be posted in the chat and pinned.

1. Everyone clones, creates a venv, and gets `pytest` passing:

```bash
git clone https://github.com/arsentis/qrpack.git && cd qrpack
python3 -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
python -m pytest -q          # must print "2 passed"
git config --global user.email "the email on your GitHub account"
```

   That last line matters: with the wrong email your commits are **not counted
   as yours**, and nobody notices until it is too late to fix.

2. Everyone confirms these function signatures. They are the seams between the
   six modules; once fixed, six people can work at once without blocking:

```python
pack_bytes(src: Path) -> bytes
unpack_bytes(blob: bytes, dest: Path) -> list[str]
chunk(blob: bytes, chunk_size: int) -> list[bytes]          # ASCII payloads
make_sheet(payloads, title, out_path, cols, scale) -> tuple[int, int]
scan_image(path: Path, cols: int) -> list[str]
reassemble(texts: list[str]) -> tuple[bytes, str, list[int]]
```

3. The payload format is frozen now and never changes:

```
QRPK1|<sha256-first-16>|<index>/<total>|<base64 chunk>
```

4. Everyone replies "ready" in the chat once `pytest` passes on their machine.
   Anyone whose environment is broken gets helped immediately — a member who
   spends day 1 fighting their venv contributes nothing on day 1.


## Day 1 — every module importable by tonight

| Who | Issue | Done when |
|---|---|---|
| Arsen | `pack_bytes` / `unpack_bytes` (tar + lzma) | round trip in memory is byte-identical |
| Aubakir | `Header` + `chunk()` + sha256 | a header parses back to the same fields |
| alikaiyr | payloads → QR images (segno) | a PNG of one code exists on disk |
| Yerassyl | `decode_one`: one QR image → text | decodes what alikaiyr produced |
| Arman | CLI skeleton + CI | green check on a pull request |
| Aslanbek | README problem and aim, `calibrate.py` stub | README explains the goal |

**End of day 1:** pack → chunk → reassemble → unpack works in memory. No paper
yet. Aubakir and Yerassyl pair on the first integration test.


## Day 2 — end to end, through a real image

| Who | Issue |
|---|---|
| Arsen | skip junk dirs, deterministic tar, path-traversal guard |
| Aubakir | sha256 verification on restore |
| alikaiyr | grid layout, uniform cells, labels, multi-sheet, PDF |
| Yerassyl | scan a whole sheet, report missing part numbers |
| Arman | wire `estimate` / `pack` / `restore` for real |
| Aslanbek | `calibrate`: payload ladder and QR version table |

**End of day 2 — this must work:**

```bash
qrpack pack ./folder -o sheet.png
qrpack restore sheet.png -o out
diff -r ./folder out        # must print nothing
```

If it does not work by the end of day 2, stop adding features on day 3 and fix
this instead. It is the demo.


## Day 3 — findings, docs, release

| Who | Issue |
|---|---|
| Arsen | README usage section with real pasted output |
| Aubakir | damage test: black out a code, assert the missing part is named |
| alikaiyr | print a sheet, photograph it, restore it, record the result |
| Yerassyl | README measured-findings section (every number with its command) |
| Arman | tag v0.1.0, CI badge, final `git shortlog` check |
| Aslanbek | `calibrate`: px/module analysis and chunk recommendation |

Leave the last 3 hours free for the submission itself. Merge nothing large in
the final hour.


## Rules for a 3-day sprint

- **Push every 1–2 hours**, not once a day. Unfinished work belongs on a branch.
- **Review within 30 minutes.** A pull request idling for 4 hours costs ~15% of
  the sprint. Still one approval, still never your own.
- **Rebase and merge only.** Squash would delete most of a 3-day history.
- **Blocked for 45 minutes? Say so in the chat.** Nobody sits stuck.
- Keep pull requests under ~150 lines so they can be reviewed quickly.


## Using the reference prototype

A working prototype exists and Arsen has it. Using it as a *guide* when you are
stuck is legitimate; pasting it wholesale is not, and it shows — the repo would
contain finished modules with no trace of anyone solving anything.

Rule: read it, close it, type your own version, and say in the pull request that
you consulted the reference. Your tests must be your own.


## Check twice a day

```bash
git fetch && git shortlog -sne --all
gh pr list --state merged --limit 50
```

Midday and evening. If someone has zero commits at the end of day 1, reassign
work that evening. By day 3 it cannot be fixed.


## Invite status

Everyone must **accept the collaborator invite** before issues can be assigned
to them — GitHub silently drops assignees who are not yet collaborators.

```bash
gh api repos/arsentis/qrpack/collaborators --jq '.[].login'   # accepted
gh api repos/arsentis/qrpack/invitations   --jq '.[].invitee.login'  # pending
bash assign-issues.sh      # re-run after each person accepts
```

## No bot authorship

Do not install the Claude, Copilot, or any other GitHub App on this repo: it
commits as `claude[bot]` and is counted as a contributor, which devalues the
graph this project is judged on. Using an assistant locally is fine — commit
under your own name and GitHub email so the credit is yours. Never leave an AI
`Co-authored-by:` trailer in a commit message; some tooling adds one silently.

Audit any time:

```bash
bash audit-authors.sh
```
