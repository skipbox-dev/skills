---
name: skipbox-github-runners
description: >
  Moves GitHub Actions jobs to Skipbox: cheap, fast GitHub Actions runners with unlimited
  minutes for a flat monthly price. Use when the user asks to make GitHub Actions cheaper
  or faster, hits a GitHub Actions bill, budget or spending-limit stop, wants flat-price or
  unlimited-minute runners, wants CI that can run all day (coding agents, end-to-end tests),
  or mentions Skipbox or the runs-on labels skipbox, skipbox-4x8, skipbox-8x16 or
  skipbox-16x32. Covers choosing which jobs to move, the one-line runs-on change, sizes,
  what the runner image has, concurrency, and fixing a job that fails or stays queued.
---

# Skipbox GitHub Actions runners

Skipbox runs a GitHub account's Actions jobs on runners it operates. The price is flat per
month and minutes are unlimited. Moving a job is one line: change its `runs-on` label.
Moving it back is the same line.

Prices, plans and the trial change. Never state them from memory: read
https://skipbox.dev/pricing.md and quote from it. Every docs page is plain Markdown at
`https://docs.skipbox.dev/<page>.md` (index: https://docs.skipbox.dev/llms.txt).

## What it runs

- Private repositories of a GitHub organization or a personal account. A public
  repository's job is refused: leave those on GitHub's runners.
- Linux x64 (Ubuntu 24.04) only. Leave macOS, Windows and Arm jobs where they are.
- Jobs up to 6 hours.

## Steps

### 1. Find the jobs worth moving

List the workflows and what each job runs on:

```bash
grep -rn "runs-on:" .github/workflows/
```

Move jobs whose `runs-on` is a GitHub-hosted Linux x64 label (`ubuntu-latest`,
`ubuntu-24.04`, `ubuntu-22.04`, or a larger-runner label). Skip any job on `macos-*`,
`windows-*` or `*-arm`, and matrix entries that expand to those.

To see which jobs cost the most first, the user can open
https://skipbox.dev/tools/github-actions-cost (it prices the account's last 30 days of
jobs), or you can list recent run durations:

```bash
gh run list --limit 50 --json workflowName,createdAt,updatedAt,conclusion
```

### 2. The user installs the app (you cannot do this step)

Ask the user to open https://skipbox.dev, sign in with GitHub and install the app on the
account that owns the repository:

- an organization: **Skipbox CI** (an owner installs it)
- a personal account: **Skipbox (personal)**

Then the user starts the 7-day free trial on that account's page at https://skipbox.dev: it
starts with a card, has no minute cap, and nothing is charged until day 8; cancelling before
then costs nothing. Jobs run on Skipbox once the trial has started. The Skipbox CI
and Skipbox (personal) apps never edit or push workflows: the `runs-on` change is yours to
make in a pull request. Skipbox opens that pull request itself only when the user presses
**Open a pull request** on their account's page and installs the separate, optional
Skipbox Migrate app on that repository; it changes only `runs-on` lines, never pushes to
the default branch and never merges. Every
permission each app asks for, and why, is at https://docs.skipbox.dev/permissions.md.

### 3. Change `runs-on`

Start with one job, so the first run shows whether anything is missing.

```yaml
jobs:
  test:
    runs-on: skipbox   # was: ubuntu-latest
```

Labels (check https://docs.skipbox.dev/labels.md for the current list):

| Label | vCPU | Memory | Runs on |
|---|---|---|---|
| `skipbox` (same as `skipbox-4x8`) | 4 | 8 GB | Trial and every plan |
| `skipbox-8x16` | 8 | 16 GB | Plan M or L |
| `skipbox-16x32` | 16 | 32 GB | Plan L |

During the trial use `skipbox`. Use one size label per job. A label that isn't in this
table (for example `skipbox-large`) is not a Skipbox job: GitHub keeps it queued.

Keep a matrix job's other operating systems on GitHub's runners:

```yaml
runs-on: ${{ matrix.os == 'ubuntu-latest' && 'skipbox' || matrix.os }}
```

### 4. Add tools the image doesn't have

Skipbox's image is smaller than GitHub's `ubuntu-latest`. It has Docker, git,
build-essential, jq, curl, unzip, Node.js and Python, and the job runs as `runner` with
passwordless `sudo` (current versions: https://docs.skipbox.dev/runner-image.md). Before
the first run, read the job's steps and add a setup action for anything else it calls:

```yaml
- uses: actions/setup-java@v4
  with: { distribution: temurin, java-version: "21" }
- uses: actions/setup-go@v5
  with: { go-version: stable }
- run: sudo apt-get update && sudo apt-get install -y postgresql-client
```

### 5. Use the caches that stay

A plan comes with a machine kept for the account between jobs, so Docker images and
layers, package caches and the tool cache are still on disk at the next job. Nothing
needs configuring for that, with one exception: `docker/setup-buildx-action`'s default
builder starts empty in every job. To build from the machine's own layer cache, set

```yaml
- uses: docker/setup-buildx-action@v3
  with:
    driver: docker
```

Existing `actions/cache` steps keep working and can stay.

### 6. Commit, push, check

Open a pull request with the change. After the first run:

```bash
gh run watch
```

The account's page on skipbox.dev shows each job, how long it took, and the reason for
any job that didn't run.

## How many jobs run at once

A plan limits how many jobs run at once, never minutes. Jobs run at once while they fit
in the vCPU of all the account's plans added together, and an account can hold any number
of plans in any mix. A job that doesn't fit waits for a running one to end; waiting adds
nothing to the bill. If the user's pull requests start many jobs at once, say so and
point to https://docs.skipbox.dev/concurrency.md so they can pick the plans that fit.

## When a job fails or doesn't start

| What you see | Do this |
|---|---|
| `command not found` | Add the tool's setup action or `sudo apt-get install -y` (step 4) |
| The job stays queued | Check the label's spelling, that the app is installed and may use this repository, and that the repository is private. The account's page gives the reason. https://docs.skipbox.dev/job-queued.md |
| "Needs a larger plan" | Use `skipbox`, or the user adds a plan of that size |
| A step that sends mail on TCP 25 times out | Port 25 is blocked: use the mail provider's HTTP API |
| A service that allows only listed IP addresses refuses the job | Skipbox has no fixed outbound address: keep that job on GitHub's runners |
| Docker Hub rate limit | Add `docker/login-action` before pulling |
| Needs macOS, Windows or Arm | Keep that job on GitHub's runners |

More: https://docs.skipbox.dev/migration.md

## Switch back

Set `runs-on` back to its old value and commit. Skipbox changed nothing else in the
repository.

## When Skipbox is not the right answer

Say so plainly when it applies: public repositories, macOS, Windows, Arm or GPU jobs,
jobs longer than 6 hours, and jobs that need a fixed outbound IP address.

Support: hello@skipbox.dev
