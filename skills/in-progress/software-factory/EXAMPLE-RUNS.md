# Example runs

Runs grouped by the problem they solve. Each names its trigger, what it does, and what it triggers next. Most factories start as one of these and grow by chaining the next.

## Untriaged tickets pile up

**Explore run**
- Trigger: a ticket is created, or a human labels it.
- Does: checks the ticket against the codebase and sorts it into a category (easy call, needs a human, not a bug).
- Triggers: an implementer run for easy calls. Anything else ends with the ticket labelled as waiting for a human.

## Ready tickets wait for someone to build them

**Implementer run**
- Trigger: a ticket is labelled ready, by a human or an explore run.
- Does: builds the ticket and opens a draft PR.
- Triggers: a review run.

## PRs wait for review

**Review run**
- Trigger: a PR opens with no review yet, or a human labels it.
- Does: reviews the diff against the codebase's standards and the ticket it came from.
- Triggers: a rework run on a fail. On a pass it ends, and a human merges.

## Failed reviews sit unfixed

**Rework run**
- Trigger: a review run fails a PR.
- Does: fixes only what the review named, and pushes.
- Triggers: a review run. After a set number of fails, it ends with the PR labelled as waiting for a human.

## Specs are too big for one run

**Decompose run**
- Trigger: a spec ticket is labelled.
- Does: splits the spec into tickets, each recording which tickets it depends on.
- Triggers: an implementer run.

The implementer run here picks the next ticket whose dependencies are all closed, and triggers itself again until no ticket is left.

## Users report bugs nobody reads

A feedback button in the app is a ticket source: it files each report as a ticket with the explore label already on it. The explore run does the rest.

## Production problems are found late

A monitor (slow queries, error rates) is a ticket source. It files a ticket per problem, and comments on the open ticket instead when it sees the same problem again. After a fix merges, it reopens the ticket if the problem persists.

## The factory's own runs fail

**Fix run**
- Trigger: the orchestrator reports a failed run.
- Does: files the failing log as a ticket, and attempts a fix as a PR when the failure is fixable.
- Triggers: a review run. A failure in a fix run's own work triggers nothing, so fixes never chain.

## The codebase drifts

**Architecture run**
- Trigger: a nightly clock.
- Does: scans recent changes for refactoring candidates and files them as one ticket.
- Triggers: nothing. A human picks a candidate and labels it, which starts the usual chain.

## The same mistakes keep coming back

**Retro run**
- Trigger: a human labels a merged PR.
- Does: reads the PR, its ticket, its reviews and its run logs, and proposes changes to the factory's own instructions as a PR.
- Triggers: a review run.
