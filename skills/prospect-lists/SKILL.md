---
name: prospect-lists
description: Build B2B prospect lists with Primavera, a database of businesses from Google Maps read with their websites (email addresses, phone numbers, websites, executives). Use when the user wants leads, a prospect list, an email list of businesses of a trade in a place ("plumbers in Lyon", "independent restaurants in Texas"), or to count, preview, export or enrich local businesses by category and area.
license: MIT
allowed-tools: mcp__plugin_primavera_primavera__get_account, mcp__plugin_primavera_primavera__find_areas, mcp__plugin_primavera_primavera__find_categories, mcp__plugin_primavera_primavera__find_networks, mcp__plugin_primavera_primavera__count_search, mcp__plugin_primavera_primavera__get_search_metrics, mcp__plugin_primavera_primavera__preview_search, mcp__plugin_primavera_primavera__estimate_list, mcp__plugin_primavera_primavera__list_lists, mcp__plugin_primavera_primavera__get_list, mcp__plugin_primavera_primavera__get_list_items, mcp__plugin_primavera_primavera__prepare_list_file, mcp__plugin_primavera_primavera__get_list_file, mcp__plugin_primavera_primavera__estimate_list_job, mcp__plugin_primavera_primavera__get_job
---

# Prospect lists with Primavera

Primavera holds the Google Maps places of France and of other countries, read with their websites. The Primavera tools
act on the user's own account: same credit balance and same prices as https://primaverapp.com. The free tools (lookups,
counts, previews, estimates, reading lists) run without asking; the paid tools (`suggest_categories`, `create_list`,
`complete_list`, `start_list_job`, `verify_list_emails`) always wait for the user's permission.

Answer in the user's language. Some texts come from Primavera in French (`selection.message`, questions, notes):
translate them.

## The path, from a request to a file

1. **Read the request.** Find the target (a trade, an activity, or what the user sells), the place, the number of rows,
   and whether only places with an email address are wanted. Ask one question only when two readings would change the
   target; otherwise go on and say in one sentence what you understood.
2. **`get_account`**: plan and balance. A `plan_required` refusal means the API needs a Pro or Agency subscription: tell
   the user (https://primaverapp.com/dashboard/billing) and stop.
3. **`find_areas`** with the place as written ("Lyon", "Gironde", "30 km around Nantes"). Take the `area` of the right
   result, or of `match` when its status is `resolved`. When several places fit, choose the obvious one or ask.
4. **`suggest_categories`** with the target in plain words, WITHOUT the place, and the area. It costs 100 credits
   (refunded when no selection comes back): tell the user before the first call. Never invent category ids. Keep the
   `search` it returns and reuse it; to change the area or a filter, edit `search` and count again, without a new
   suggestion. Call it again only when the target itself changes. A `clarification_needed` refusal (refunded) carries a
   question in `error.details`: ask it, then call again with the answer.
5. **Figures, free**: `get_search_metrics` (total, with email, phone, website) or `count_search`; `preview_search` for a
   few sample places (contact details stay hidden until a list is created). When a figure has `exact: false`, say
   "≈ N". Never call the result exhaustive.
6. **`estimate_list`** with `search`, `rows` and, when only places with an email are wanted, `withEmail: true`. Tell the
   user: the rows, the cost in credits and the balance after. Agency accounts export new places without credits under
   the fair use limit (`included: true`).
7. **`create_list`**, once the user agreed: `name`, the same `search`, `rows` and `withEmail`, `maxCredits` = the `cost`
   of the estimate, and a new `idempotencyKey`. While the list says `creating`, call `get_list` every 10 seconds or more.
8. **Download.** `prepare_list_file` (`format`: `csv` for every column, `xlsx` to choose columns) with a new
   `idempotencyKey`, then `get_list_file` until `status` is `ready`. The `url` is valid 120 seconds: in Claude Code,
   download it at once (`curl -L -o "<list name>.csv" "<url>"`); elsewhere, give it to the user and say it expires in
   two minutes (call `get_list_file` again for a new one). For a quick look, `get_list_items` with a small `limit` (50 or
   less) and a few `columns`.

## Enrich a list (paid, optional)

- `estimate_list_job` then `start_list_job` with the same `tool`, `question`, `criterion` and `scope`, and
  `maxCredits` = the estimate's `maxCredits`: `executives` (company executives), `summary` (a short summary of each
  website), `question` (a yes/no question asked to each website), `filter` (a criterion judged on the data). The job runs
  in the background: follow it with `get_job`; results appear in `get_list_items`. Never promise a rate of executives
  found.
- `verify_list_emails`: `maxCredits` = `progress.emailCheck.unverified` of `get_list`; 1 credit per address that gets a
  verdict, the rest is returned.

## Money rules

- Every paid call needs `maxCredits` (the most it may spend, never exceeded) and `idempotencyKey`, a new UUID v4 per
  operation. Without them nothing runs. Make one with `uuidgen` (or `python3 -c "import uuid; print(uuid.uuid4())"`), or
  write a random version 4 UUID yourself.
- To retry after an error or a timeout, send the same `idempotencyKey` with the same arguments: the saved result comes
  back and nothing is charged twice. Never reuse a key for another operation.
- Tell the cost before any paid call, and wait for the user's go.

## When a call is refused

Every refusal carries `error.code`, `error.message` and `error.hint`: do what the hint says. Never retry in a loop; after
`rate_limited`, wait the seconds given. Common codes: `insufficient_credits` (lower the rows or ask the user to top up),
`fair_use_limit` (Agency: see `get_account`), `max_credits_too_low` (estimate again, then send the new cost),
`target_unresolved` (call `suggest_categories`), `too_many_active_jobs` (wait for the running job).

Renaming, merging and deleting lists are not available here: the user does it at https://primaverapp.com.
