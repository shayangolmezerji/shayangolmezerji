Three of the four repositories below have a section listing what their own tests
have not proven. The fourth has a paragraph about a deployment that failed.

Shayan Golmezerji. Istanbul. Roughly three years of web development, AI and
blockchain work, and these four are the ones I would defend line by line in a
follow-up call.

## Repos

[menu-events](https://github.com/shayangolmezerji/menu-events): event-sourced
restaurant menu over FastAPI and PostgreSQL. The append-only event log is the
authority, and the read model is rebuilt rather than patched when it goes wrong.
On a machine with no database `pytest` reports `100 passed, 18 skipped`, and the
18 skips are the PostgreSQL tier, which passed against 16.15 in a container on
2026-09-24. A green run with no database still proves nothing about the SQL in
`store/postgres.py`.

[deadman-ssh](https://github.com/shayangolmezerji/linux-recovery): a bash script
over coreutils, `awk` and `iproute2`, with nothing to compile. Arm it before a
firewall change and it restores the ruleset it captured if your SSH session
leaves the socket table or the countdown runs out. 27 groups of plain-bash
checks, 246 of them on a box without shellcheck and 256 with it. A drop rule
that leaves the socket `ESTABLISHED` is the case it was written for and the
probe does not see it, so that README says to arm with a countdown you can
actually sit through.

[pr-summarizer](https://github.com/shayangolmezerji/pr-summarizer): reads a
unified diff with the standard library's `ast` and reports which symbols
appeared, vanished, moved between files, or had a signature widened, with the
call sites counted from the diff. 103 tests at 97% statement coverage, and
`pip show` reports an empty `Requires:` field. A language model is an optional
renderer bolted on top, unconfigured by default, and the structural brief does
not need one.

[devlog](https://github.com/shayangolmezerji/devlog): Astro site, three
long-form posts about decisions in the repos above, no JavaScript shipped to the
reader. Its deploy workflow was red three times on `Ensure GitHub Pages has been
enabled`, and the fix was making the repository public plus one POST to the
Pages endpoint, not a line of the workflow.

## Writing

<https://shayangolmezerji.github.io/devlog/> is live. Run `36276843655`, on
`main` at `c06734c`, green on both jobs, index and all three posts at HTTP 200.

## This account, rendered

![Contribution grid with a snake drawn along it](https://raw.githubusercontent.com/shayangolmezerji/shayangolmezerji/output/github-contribution-grid-snake.svg#gh-light-mode-only)
![Contribution grid with a snake drawn along it](https://raw.githubusercontent.com/shayangolmezerji/shayangolmezerji/output/github-contribution-grid-snake-dark.svg#gh-dark-mode-only)

![Contributions calendar, and the one language the card counts](https://raw.githubusercontent.com/shayangolmezerji/shayangolmezerji/main/github-metrics.svg)

Two workflows in this repository draw those from this account's public activity,
on GitHub runners, so the rendering never leaves GitHub. `WIDGETS.md` holds what
was tried and rejected, with the command that re-checks each figure. The card
reads `1 Language` because `lowlighter/metrics` counts what this account
authored, and that is a small set so far: it is not a language mix, and the
calendar is sparse for the same reason. A scheduled workflow only refreshes
while the repository sees activity, so both images can go stale without
breaking.

## Contact

Open an issue on the repo your question is about. The code and the measurements
are already sitting in it, and that is a better first message than a greeting.

- Email: <mailto:mastershayan@proton.me>
- Telegram: <https://t.me/SHAYANGOLMEZERJI>
