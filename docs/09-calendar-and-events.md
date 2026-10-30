# 09 · Calendar and events

## Date windows

- Quests (and other content such as eggs and shop items) have `start_date` / `end_date`
  strings, `YYYY-MM-DD HH:MM:SS`, read as **UTC** with no timezone handling (CONFIRMED).
- The client stores them as **signed 32-bit Unix seconds**. Valid range ends at
  2038-01-19 03:14:07 UTC. A safe "permanent" end is `2037-12-31 00:00:00`. Dates like
  2099 overflow and the content disappears or the screen loops (CONFIRMED, observed).
- Active means `start <= now < end` (end exclusive). An **empty** date is not "forever": it
  converts to -1 and an empty end fails the check (CONFIRMED).
- `now` is the server time from the latest response carrying it (`serverTS` in login/home,
  `server_time` in the resource list). Without it the client uses 0 (1970) and anything
  starting later is invisible (CONFIRMED).

## Quest types and pages

- `world_map_page` 1..9 are the story pages; page **0** is the **event page** (its own map,
  reached by the *Events* button).
- `type 1` = normal quest; `type 4` = special event (enters the event page directly when its
  window contains `now`). Other types exist on page 0 in the original data (daily dungeons,
  trials, ranking events…) and are not documented here.
- The original data placed hundreds of timed type-1 quests on page 0 (dungeons and "trials")
  alongside a handful of type-4 events.
- If page 0 has no active quest at all, the *Events* button just reloads the current map
  (observed).

## Difficulty groups (event areas)

- Event quests list their areas in **groups of three consecutive area IDs** sharing the same
  `scenario_id` (for example three stages of "Beginner", three of "Intermediate", three of
  "Advanced"). `scenario_id` is the grouping key in memory (CONFIRMED).
- The client groups areas by `quest_id` and changes of `scenario_id`, and some paths compute
  stage IDs as **base + index** (base = the group's first area). Serving only some stages of
  a group leaves gaps those lookups will hit (CONFIRMED that the lookups exist).
- In the original encounter data, often only the **first** stage of each group has a complete
  encounter table; the other stages define only the boss round. How the original server or
  client filled the remaining rounds is UNKNOWN.

## A warning for reconstructions: the event page freeze

A private restoration served original event rows with a valid date window and observed the
client **freeze while loading the event page** (render thread blocked, no crash, no network
request). Tried and ruled out as the sole cause: a missing story-scene archive, missing sibling
stages of the difficulty groups, the absence of a type-1 quest on page 0, and the reward card
art. The cause is still UNKNOWN. If you get further, a native backtrace of the blocked render
thread is the most useful evidence.

## Seasonal events (a reconstruction policy)

The original events were one-off server schedules; their full calendar is lost. A practical
policy for a restoration (RECONSTRUCTED):

- story campaign and compatible historical events: always available (window up to 2037-12-31);
- seasonal events recur yearly, e.g. Christmas `Dec 1 → Jan 1`, Easter-relative
  `Easter − 13 days → Easter + 5 days` (the 2016 forum timeline fits this window);
- keep the original dates untouched in an archive and apply the windows only to the served copy.

## Login bonus calendar

The calendar button on the home screen opens the **login bonus** calendar (a local table of
daily rewards). Collecting goes through the gift API. The original reward schedule is in the
client's data; the server decides which day the player is on.
