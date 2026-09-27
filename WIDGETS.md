# Profile graphics

What renders on the profile page, what was tried and taken back out, and the
command that re-checks each figure below.

Everything here was measured on 2026-09-26 and 2026-09-27 from one machine
against the live `shayangolmezerji` account. The run ids and byte counts are
measurements of runs that already finished, not predictions of what the crons
will produce next. Each figure that came from a run carries the read-only call
that proves it, and every one of those calls was run from this checkout on
2026-09-27. Three of them did not answer the way an earlier draft of this file
claimed, so those claims are corrected in place and each correction names the
check that settled it. That call, not the number, is what a reader should trust.

## The shape of it

```
 snake.yml
   Platane/snk/svg-only            -> dist/github-contribution-grid-snake.svg
                                      dist/github-contribution-grid-snake-dark.svg
   crazy-max/ghaction-github-pages -> `output` branch, both files at its root
   cron 17 4 * * *

 metrics.yml
   lowlighter/metrics              -> github-metrics.svg on `main`
   cron 43 5 * * *

 README.md
   img src -> raw.githubusercontent.com/shayangolmezerji/shayangolmezerji/output/github-contribution-grid-snake.svg
   img src -> raw.githubusercontent.com/shayangolmezerji/shayangolmezerji/main/github-metrics.svg
```

Neither renderer is a third-party image endpoint. Both actions run on this
account's own Actions minutes and their output is served from inside this
repository, so a visitor's browser talks only to GitHub. That is the reason
these two exist here and the hosted cards do not.

## Shipped

### `snake.yml`

Commit `24ac369`, subject "Render the profile graphics on this account's
runners". Workflow run `36310443557`, conclusion `success`, first run, green.

It renders `github-contribution-grid-snake.svg` and
`github-contribution-grid-snake-dark.svg` (the dark one with
`palette=github-dark` on its output line) into `dist/`, then publishes `dist/`
to the `output` branch with `keep_history: true`. Scheduled `17 4 * * *`, plus
`workflow_dispatch`. `permissions: contents: write`, nothing else.

```bash
gh run view 36310443557 --repo shayangolmezerji/shayangolmezerji \
  --json conclusion,headSha,workflowName
gh api repos/shayangolmezerji/shayangolmezerji/branches/output --jq .name
curl -s -o /dev/null -w '%{http_code} %{url_effective}\n' \
  https://raw.githubusercontent.com/shayangolmezerji/shayangolmezerji/output/github-contribution-grid-snake.svg
curl -s -o /dev/null -w '%{http_code} %{url_effective}\n' \
  https://raw.githubusercontent.com/shayangolmezerji/shayangolmezerji/output/github-contribution-grid-snake-dark.svg
```

### `metrics.yml`

Workflow run `36310441387`, conclusion `success`, first run, green. It commits
`github-metrics.svg` to `main`: 132,818 bytes at the commit that run wrote.
Scheduled `43 5 * * *`, plus `workflow_dispatch`.

The action writes through the GitHub API rather than a checkout, which is why
there is no `actions/checkout` step in that file. `filename` is pinned to
`github-metrics.svg` because the README links that path, and `user` is set
explicitly because the action otherwise resolves the account from `GET /user`,
which for a workflow token is not necessarily the repository owner.

```bash
gh run view 36310441387 --repo shayangolmezerji/shayangolmezerji \
  --json conclusion,headSha
gh api repos/shayangolmezerji/shayangolmezerji/contents/github-metrics.svg \
  --jq .size
```

### Concurrency

`snake.yml` declares `concurrency.group: publish-output-branch` with
`cancel-in-progress: false`. `metrics.yml` declares no `concurrency:` block.

```bash
grep -n -A2 '^concurrency:' .github/workflows/*.yml   # one match, in snake.yml
```

The group therefore has one member, not two, and its comment explained the guard
by `snake.yml` and `trophy.yml` both committing to `output`. `trophy.yml` is gone.
The comment was rewritten in the same commit that deleted the workflow: the guard
is now stated for what it actually protects, which is this workflow's own next
run, since a dispatch overlapping the schedule would have two runs pushing the
same branch and the publisher does not force.

`metrics.yml` deliberately still has no block. It writes `github-metrics.svg` to
`main` through the API, not to `output`, so it is not the second non-forcing
publisher the group was named for, and putting it in the same group would make
the two renders queue behind each other for no reason. Adding the group there
would be the wrong fix, and that is settled by reading the two `target` settings
rather than by argument:

```bash
grep -n 'target_branch\|filename' .github/workflows/*.yml
```

## Pins

Each action is pinned to a full commit sha with its tag in a trailing comment.
All three lines are in the two workflow files:

| `uses:` | sha | comment |
|---|---|---|
| `Platane/snk/svg-only` | `d8f6715049803e982ee5ff501b6b9b7d5deeb09b` | `# v3.5.0` |
| `crazy-max/ghaction-github-pages` | `1d6ee9b181a81033a16bd707a1401afa978daab4` | `# v5.0.0` |
| `lowlighter/metrics` | `65836723097537a54cd8eb90f61839426b4266b6` | `# v3.34` |

Re-checking a pin means asking GitHub whether that tag's object is that sha:

```bash
gh api repos/Platane/snk/git/ref/tags/v3.5.0 --jq '.object.sha'
gh api repos/crazy-max/ghaction-github-pages/git/ref/tags/v5.0.0 --jq '.object.sha'
gh api repos/lowlighter/metrics/git/ref/tags/v3.34 --jq '.object.sha'
```

Two traps in that. Add `--jq .object.type` to the same call: if it answers `tag`
rather than `commit`, the ref points at an annotated tag object and its sha is
not the commit sha, so the comparison has to dereference one more hop with
`gh api repos/<owner>/<repo>/git/tags/<object-sha> --jq .object.sha`. And
`Platane/snk/svg-only` is a subdirectory of `Platane/snk`, not a ref namespace:
that repository tags its root, so there is no `svg-only/v3.5.0` to look up. A
checker that builds the ref name out of the whole `uses:` path reports a false
mismatch on that row.

Run on 2026-09-27, all three answered `commit` and returned exactly the sha in
the table above, so the pins and their comments agree. The annotated-tag trap is
live in the other direction here: none of these three refs is a tag object, so a
reader who copies the first command and gets `commit` back does not need the
second hop at all.

Every input those workflows pass was checked against the action's own definition
file at the pinned sha, not against its README. Reproduce it for two of the
three:

```bash
gh api 'repos/Platane/snk/contents/svg-only/action.yml?ref=d8f6715049803e982ee5ff501b6b9b7d5deeb09b' \
  --jq .content | base64 -d | sed -n '/^inputs:/,/^[^ ]/p'
gh api 'repos/lowlighter/metrics/contents/action.yml?ref=65836723097537a54cd8eb90f61839426b4266b6' \
  --jq .content | base64 -d | grep -E '^  (token|user|filename|plugin_isocalendar|plugin_languages):'
```

Both greps assume the shape a normal `action.yml` has: an `inputs:` map at column
zero, its keys indented two. If either comes back empty, print the block and read
it rather than concluding the input does not exist.

## Tried and rejected

### `ryo-ma/github-profile-trophy`

Written, committed to a branch named `trophy-no-token`, pushed, run twice, then
removed. The two runs are the record of the attempt; the commit it was tried in
is still in this checkout:

```bash
tail -5 .git/logs/HEAD                       # the branch, the commit, back to main
git log --format='%h %ad %s' --date=short -1 df49fed
ls .github/workflows/                        # two files; no trophy workflow
```

`df49fed`, "Try the trophy render without a token", is reachable from no ref in
this checkout, only from the `main` reflog, so a `git gc` here eventually prunes
it. The run ids below are the durable record.

Run `36310445154` passed `token: ${{ github.token }}`. Run `36310565546` passed
no token override and took the action's default. Both stopped at the
`Render trophy card` step with the same three lines:

```
Error fetching user info for username: shayangolmezerji
Failed to fetch user info. Check token, username and rate limits.
Error: Max retries (2) exceeded.
```

```bash
gh run view 36310445154 --repo shayangolmezerji/shayangolmezerji --log-failed
gh run view 36310565546 --repo shayangolmezerji/shayangolmezerji --log-failed
```

What the logs carry is a failed fetch of user info under two different token
configurations. That is the whole measurement. The next step, "so the card needs
a personal access token", is an inference from those two runs and not something
either log states: repeat it as an inference or go read the action's own docs on
what it fetches and with what. The decision did not depend on settling it. The
reason these graphics exist on this account's runners is to stop depending on
third-party renderers and the credentials they ask for, so a card that wants a
PAT was removed rather than left red or wired to one.

### Removed alongside it

`readme-typing-svg`, the `komarev.com` profile-view counter,
`github-readme-stats` streak card, `github-readme-stats` top-langs card, and
`skillicons.dev`. Same reason: third-party renderers in the request path of
every visitor who opens the page. The streak and view counters have a second
problem, which is that they render numbers that are not about the work. The
previous version of this README had all of them; `README.md` here has none.

## What the metrics card actually says

Verified by pulling the strings out of the `github-metrics.svg` served from
`main`. The card reads:

```
1 Language
Most used languages
HTML
Contributions calendar
These metrics do not include all private contributions
```

```bash
curl -s -o /tmp/m.svg https://raw.githubusercontent.com/shayangolmezerji/shayangolmezerji/main/github-metrics.svg
for s in "1 Language" "Most used languages" "HTML" "Contributions calendar" "do not include all private"; do
  printf "%-32s %s\n" "$s" "$(grep -c "$s" /tmp/m.svg)"
done
```

Each of the five answers `1`. An earlier version of this section gave
`grep -o '>[^<>]*</text>'` as the re-check command, which returns nothing and
proves nothing: the card is HTML wrapped in an SVG, so it has no `<text>`
elements at all. `grep -o '<[a-zA-Z]*' /tmp/m.svg | sort | uniq -c` shows what is
in there instead, 581 `path` and 35 `div` among them, which is why the strings
have to be grepped rather than parsed out of a text layer.

One language, HTML, is not a language mix, and the card should be described as
what it renders rather than as evidence of a stack. `lowlighter/metrics` computes
languages from commits the account authored, which was the reason to expect a
different answer: `3x-ui`, a fork on this account, is Go and tens of megabytes,
and the worry was that it would swamp the card. Checked, and it did not
materialise. What is left is a sparse card because the authored work on this
account is sparse. The footer line about private contributions is the card's own
wording and it stays in the record because it bounds what the calendar means.

## What this checkout can and cannot prove

The tracked tree here is `README.md`, `WIDGETS.md`, `index.html`, `profile.jpg`
and two files under `.github/workflows/`. `main` and `origin/main` both sit at
`24ac369`. So the commit that carried the workflows, the commit the trophy was
tried on and the text of both workflow files are checkable from disk. Not one
output is: there is no `output` ref, no `github-metrics.svg` in the working
tree, and no `resume.pdf`. Anything about those paths has to go through
`gh run view`, `gh api` or `curl`, which is why every such claim above carries
its own command rather than a file name.

## Schedule behaviour

A public repository with no activity for 60 days has its scheduled workflows
disabled automatically. These two crons are not a substitute for pushing: a quiet
repository stops producing refreshed graphics, and a stale strip is worse than an
absent one because it implies activity that is not happening. The same page also
says a scheduled event can be delayed under high load, that high load includes
the start of every hour, and that scheduled workflows only run from the default
branch, which is why both files here offset their minutes from the hour.

Source, quoted rather than paraphrased, from the *Events that trigger workflows*
document under `schedule`:

> In a public repository, scheduled workflows are automatically disabled when no
> repository activity has occurred in 60 days.

`https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows#schedule`

An earlier draft of this section cited `workflow-syntax#jobsjob_idtriggersschedule`
instead. That anchor does not exist in the page it names, and the sentence is not
on that page at all: `grep -c 'jobsjob_idtriggersschedule'` over the downloaded
`workflow-syntax` HTML returns `0`, while the same download carries the correct
`id="schedule"` heading on the page above. Check before repeating either:

```bash
curl -s https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows \
  | grep -oE 'In a public repository[^.]*\.'
gh run list --repo shayangolmezerji/shayangolmezerji --workflow snake.yml \
  --limit 3 --json status,conclusion,createdAt
gh run list --repo shayangolmezerji/shayangolmezerji --workflow metrics.yml \
  --limit 3 --json status,conclusion,createdAt
```

The `grep -oE` pattern stops at the first period on purpose. The page embeds its
own body three times over, once as HTML and twice as escaped JSON, so a pattern
that runs to the next `<` prints the sentence once and then four hundred
kilobytes of escaped markup beside it.

## Comments that described a third workflow

Three comments in the two workflow files named the trophy card that is no longer
rendered: `snake.yml` justified the concurrency group and `keep_history` by it,
and `metrics.yml` called its own cron the middle of "three renders" running
"before the weekly trophy card". All three were rewritten in the commit that
deleted `trophy.yml`, so this section is about what the files now claim rather
than what they used to.

The `keep_history` comment now quotes the action's own input table at the pinned
sha, which reads "Create incremental commit instead of doing push force", because
that is the one claim in it that a reader can check without a runner:

```bash
gh api 'repos/crazy-max/ghaction-github-pages/contents/README.md?ref=1d6ee9b181a81033a16bd707a1401afa978daab4' \
  --jq .content | base64 -d | grep -A1 keep_history
```

## The other thing this repository serves

`index.html` is tracked here and this repository has its own Pages site, `legacy`
build type, status `built`:

```bash
gh api repos/shayangolmezerji/shayangolmezerji/pages \
  --jq '{build_type,status,html_url}'
```

It answers at `https://shayangolmezerji.github.io/shayangolmezerji/` and serves
that 11-line file, whose title is `Resume` and which embeds `resume.pdf`. No
`resume.pdf` is tracked here, and the served path confirms it:

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://shayangolmezerji.github.io/shayangolmezerji/
curl -s -o /dev/null -w '%{http_code}\n' https://shayangolmezerji.github.io/shayangolmezerji/resume.pdf
ls resume.pdf
```

Those two answered `200` and `404` on 2026-09-27. An earlier draft of this file
probed `https://shayangolmezerji.github.io/resume.pdf`, which is not this site's
path and 404s for a second reason. So the profile Pages URL is a resume viewer
for a file that is not in the repository, and it is unrelated to the two graphics
above. Adding the PDF or removing the viewer are both answers; leaving it is the
one that is already true, and it is the owner's call, not this file's.
