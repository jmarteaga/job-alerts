# LA28 Hospitality — daily lowest-price monitor

A scheduled Routine checks the LA28 official hospitality store once a day and reports the
lowest-cost ticket/hospitality packages available for a fixed set of URL selections. Each
run writes a dated report here (`ticket_alerts/YYYY-MM-DD.md`), keeps rolling state in
`../ticket_alerts_state.json`, and (when new data is retrievable) creates a Gmail draft to
the owner.

## What it watches

- **URL:** https://hospitality.la28.org/en/products?events=GAR%2CATH%2CSWM&dates=&groupSize=4
- **Events:** `GAR`, `ATH`, `SWM` (the `events` URL parameter)
- **Dates:** all (the `dates` parameter is empty)
- **Group size:** `4` (the `groupSize` parameter)

To watch a different selection, edit the `url` and `selections` in
`ticket_alerts_state.json` and update the Routine prompt to match.

## What each run does

1. Fetches the monitored URL and extracts every product matching the current selections
   (package name, event, date/session, price, per-person vs. per-package).
2. Identifies the lowest-cost option(s) and compares against the previous run recorded in
   `ticket_alerts_state.json` to flag **price drops**, **new lows**, and **sold-out /
   removed** packages.
3. Writes `ticket_alerts/YYYY-MM-DD.md` with the ranked cheapest options and any changes.
4. Updates `ticket_alerts_state.json` (`lowest_seen`, `listings`, `history`).
5. Creates a Gmail draft to the owner summarizing the cheapest options and changes.
6. Commits the report and updated state.

## Schedule

Runs daily in the morning (US Pacific) as a Claude Code Routine (fresh session per fire).

## ⚠️ Network egress requirement

`hospitality.la28.org` must be reachable from the environment that runs this job. This
environment's egress policy currently **blocks** that domain (`EGRESS_BLOCKED`), the same
restriction that has intermittently broken the weekly job-alert runs. Until the domain is
allow-listed in the environment's network settings, each run records the block in its
dated report and skips the email draft (no live prices can be retrieved). Once access is
enabled, the job resumes reporting real prices automatically — no code change needed.
