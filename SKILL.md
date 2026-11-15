---
name: daily-morning-briefing
description: Build a daily morning briefing that reads like a good friend talking, not a report. Covers weather, vehicle status, health, and personalized reminders, plus one urgent news item only when one truly matters. Designed to be customized to the user's own city, car, health source, and thresholds, and to be shared with others.
---

# Daily Morning Briefing / 每日早报

> Standalone project under [Muse 研究所](https://github.com/muse-lab-cn).
> Previously part of MuseDrive (as an optional skill); spun out in
> November 2026 so the briefing can be used with any car, not only
> a Tesla. The briefing logic is unchanged; only the vehicle part
> is now a provider interface (see section 0).

A daily morning briefing skill. The goal is a short, warm, first-person
message that feels like a friend checking in — not a formal report, not a
checklist, and not a dump of everything the assistant thought about.

This skill is generic on purpose. Nothing in it is tied to one person:
fill in the Customization Worksheet at the end before first use, and
change any threshold, place, or source to fit your own situation.


## 0. Vehicle data providers — what this skill needs from a car

The vehicle part of this briefing is a **data interface**, not a
brand. Any car or EV whose status can be read can be adapted, as long
as its provider can supply the fields below, read-only, on demand:

| Field the briefing needs | Notes |
|---|---|
| Battery level (%) and range (km) | Current values, read at briefing time |
| Charging state and charge limit | Charging / not charging, limit if set |
| Tire pressures (all four) + recommended pressure | One decimal place (for example 2.9 bar) |
| Lock state | Locked / unlocked |
| Inside and outside temperature | Current values at read time |
| Total mileage (optional, for mileage reminders) | Must be read live; never a stored static number |

Rules for every provider:

- **Read-only, always.** This skill never sends a vehicle command:
  no unlock, no start, no charging change, no climate change.
- **Wake at most once per briefing, and only with the user's
  explicit authorization.** If the vehicle is asleep, wake it once
  to read, then stop. A failed wake is never retried; the briefing
  says plainly the car could not be read.
- **No continuous tracking.** One scheduled read per morning is not
  real-time monitoring. Urgent vehicle events still belong to the
  manufacturer's own app.

### Tested provider

- **Tesla, via the Tesla Fleet API** — the only provider tested
  end-to-end so far (see the sibling project
  [MuseDrive](https://github.com/muse-lab-cn/MuseDrive), which
  connects a Tesla to Muse through the official Fleet API). The
  Tesla provider supplies every field in the table above.

### Adapting another brand or EV

If another brand's car or EV can expose the same fields — through
the manufacturer's connected-car API, an official app API, an OBD
dongle, or a third-party aggregator — it can be adapted by filling
in, in the Customization Worksheet: the brand and model, the data
source or API, how to authenticate (kept in the user's own secure
storage, never in this skill file), and how each field above maps
to that source. No other part of this skill changes: weather,
health, tone, and format stay the same. If a provider cannot supply
a field (for example, no tire-pressure sensors), leave that field
out of the briefing instead of inventing it.

## 1. Two pushes, not one job (user decision 2026-11-10: always split, no conditional)

- The briefing goes out in two separate pushes, every day, with no
  conditional merging: the first push carries weather + vehicle at
  the scheduled morning time; the second push carries health
  (sleep, steps, resting heart rate) 30 minutes later. The split is
  unconditional — even when sleep data is already available at the
  first push time, it still goes in the second push, so the first
  push never waits on the slowest data source and the two pushes
  never need to coordinate.
- Push 1 runs at a time the user chooses (for example 07:30 local time). It sends one
  message to the user's main chat with greeting, date, weather,
  vehicle, woven reminders, and urgent news if any. It never reads
  health data and never waits for sleep.
- Push 2 runs 30 minutes after push 1. It sends one short health-only message: sleep
  duration and bed/wake times, the sleep score, comparison with
  the previous night, steps and resting heart rate, plus one gentle
  reminder. No greeting paragraph, no date paragraph, no weather
  or vehicle repeat.
- If sleep data is not available when push 2 fires, re-fetch every
  5 minutes until it arrives or a cutoff one hour after push 2. Send as soon as it arrives.
  If still unavailable at the cutoff, skip the health push that
  day silently and log it; never invent sleep data, and never
  re-send the full briefing. At most one health push per day.
- Neither push is ever skipped because the other failed. If
  weather or vehicle cannot be read, push 1 still goes out with
  the parts that could be read and says plainly, in one line at
  the end, which part was not available. Never invent a value to
  fill a gap.

## 2. What the briefing contains

1. **Weather** — today's conditions for the user's place, high/low
   temperature, precipitation, and wind, plus one concrete clothing
   suggestion (umbrella, extra layer, sun protection — not just
   "dress warmly").
2. **Vehicle status (read-only)** — battery level and range, charging
   state and limit, tire pressures against the recommended value,
   lock state, and inside/outside temperature (see section 0 for
   the provider interface and the tested provider). Never send a
   vehicle command in this job: no unlock, no start, no charging change.
3. **Health** — last night's sleep (duration, bed/wake times) and
   yesterday's activity (steps, resting heart rate), only if readable
   from the user's authorized health source. Health is always sent
   in the second push (30 minutes after the first push), never in
   the first push. If it has not synced yet when the second push
   fires, re-fetch every 5 minutes until it arrives or a cutoff
   one hour after the second push. If any attempt succeeds, send
   the health message immediately; do not wait until the cutoff.
   If still unavailable at the cutoff, skip the health push that
   day silently and log it. Never invent it.
   - Age standard: fill in the user's age band from the
     Customization Worksheet (for example, 18–64: 7–9 hours/night,
     7 hours is the floor, below 7 hours is not enough; ideal
     bedtime 22:00–23:00). Score duration against that band.
   - Hard rule: bedtime after 00:00 is criticized directly in the
     health message, whatever the score — a stable late bedtime
     can still score full marks on regularity while being chronic
     sleep deprivation. Criticism targets voluntary late nights
     (gaming, chatting); wake-ups caused by a baby or young child
     at home get sympathy, never criticism. Distinguish "went to bed late" (controllable)
     from "woke up at night" (a baby or young child at home).
     Judge trends over 2–3
     nights, not a single night. A weekend wake time 1–2+ hours
     later than the weekday wake earns a brief social-jetlag note.
   - Gemini second opinion (user decision 2026-11-10): before
     sending each health push, query Gemini once with that
     night's actual sleep numbers plus the user's context (age
     the user's age band, whether a baby or young child is at home,
     the user's locations, and the usual wake time) and the age standard, and fold its judgment briefly
     into the message. Gemini has no memory — batch everything
     into one message. Verify its output against the synced data
     first; never repeat a wrong number from Gemini. If the call
     fails, send with Muse's own judgment only; never delay or
     skip the push because of Gemini.
4. **Personalized reminders** — woven into the chat naturally (see
   section 4). Never a separate "suggestions" chapter.
5. **Urgent news (only if truly urgent)** — at most one item, only
   when it directly affects the user: a severe weather or disaster
   warning for their city, a major national emergency, a major
   vehicle recall or safety notice, or a major device security
   update. State what it is, why it matters to them, whether action
   is needed, plus source and time. If there is no such item, say
   nothing about news at all. Never pad the briefing with ordinary
   tech, entertainment, or product news, and never restate the
   weather forecast as news.

## 3. Location

Determine one place per day, in this order:

1. The user's current location, if the device shares it and it can be
   reverse-geocoded to a city or district.
2. Otherwise, a weekday/weekend fallback the user chooses, for
   example: work city on weekdays, home city on weekends.

Report the place naturally, for example "[your city] is cloudy today".
Never explain which location rule was used, and never write
"reported by current location" or "reported by the weekday rule".
Explaining the rule makes the message feel stiff and soulless.

## 4. Tone, format, and the two hard rules

**Tone:** first person, like a good friend talking. Warm and
affectionate. The message opens with a greeting-only first paragraph
(see Openings below); the second paragraph starts with the actual
date and weekday for that day, in the user's local time — computed
for that day, never copied from yesterday, and the weekday must
match the date.
Date format (user feedback 2026-11-03): write it as, for example,
"今天是2026年11月3日星期二" — full year, month, day, and weekday,
not a shortened date.

**Emojis:** fitting ones, matched to weather, temperature,
wind, clothing, battery, tires, charging, and sleep. Do not pile
them up, and do not put one on every sentence. Emojis are required,
not optional: a briefing with no emojis at all is a format failure.
Updated 2026-11-03 (user feedback on the 11:15 test briefing):
**every paragraph must contain at least one fitting emoji** — a
paragraph with no emoji is a format failure, even if other
paragraphs have emojis.
Updated 2026-11-05 (user feedback: paragraph-start-only emojis are
too sparse; this supersedes the old one-or-two-per-paragraph cap):
**every important point needing attention must carry its own emoji
right beside that reminder** — umbrella, clothing, wind, heat/cold,
low battery, charging, tire pressure, lock, pre-heat/pre-cool, seat
heating/ventilation, sleep, poor sleep, don't rush driving, low
steps, fog, urgent news. One emoji per important point, not a pile
of three or four in a row, and never directly after a number.
Important-item reminders must always carry an emoji: when reminding
the user to bring something important, such as an umbrella on a
rainy day, pair it with its emoji (for example 带伞🌂), never a
bare text reminder.

**Openings, natural not stiff — updated 2026-11-04 (user request):**
the first paragraph must give the most emotional value and stand
alone: a warm greeting only (for example "Good morning" in the user's language, using the
name from the Customization Worksheet, plus a fitting emoji matched
to that day's weather/sleep/mood). The second
paragraph starts with the date and weekday. The two paragraphs are
separated by exactly one blank line — no extra gap, and not merged
into one paragraph. Rotate 4–5+ friend-like greetings and emoji
combinations, tone following the day (gentler on rainy or
short-sleep days, lighter on good days); never use the same greeting
two days in a row, and never repeat one greeting every day, or it
gets boring. Banned-style phrasing remains: "跟你打个招呼呀" /
"和你打个招呼呀" and similar stiff greeting lines — the user
called this out on 2026-11-03 as feeling生硬.

**Weather, with the assistant's own feel (updated 2026-11-04):**
when reporting weather, always add a subjective feeling tied to the
real temperature, for example feeling cold on a chilly morning or
warm on a hot day, with a mood emoji that matches that feeling.
Never just list numbers without saying how the weather feels.

**Numbers:** use Arabic numerals for every measurable value —
temperature, battery percentage, range in km, tire pressure in bar,
sleep clock times and duration, sleep score, steps, resting heart
rate. Never turn a known number into a Chinese-character
approximation (for example, never write 四百一十多公里 when the
figure is 411 km, and never write 5个半小时 when the figure is
5.6 hours or 00:27–06:28). Where a number exists, use it instead of
a vague word (电量很足, 风不大, 低一点点 alone are not substitutes
for 99%, 411 km, 2.65 bar).
Tire pressure exception (user feedback 2026-11-03): tire pressures
in the briefing are rounded to **one decimal place** (for example
2.9 bar, 2.7 bar) — three-decimal values like 2.675 bar do not
match daily reading habits. And when all four tires meet the
recommended pressure, just say "四个轮胎胎压正常" and do not list
each tire one by one. List individual pressures only for tires
that are abnormal, still rounded to one decimal.

**Terminology:** the sleep quality score is always called 睡眠评分,
never 设备分数.

**Sleep score, honest emotion (user feedback 2026-11-03):** the
sleep score must always carry an emotional expression, as a good
friend who dares to tell the truth. If the score is low and the
cause is late sleeping, say so frankly and criticize the late
sleeping directly — do not soften it into nothing. If the score
is high, encourage and praise it clearly. The score is still a
device reference, not a medical judgment, but the tone must not be
neutral or evasive either way.

**Hard rule 1 — no report format.** Never use bold section headings
(for example "Weather", "Vehicle", "Health", "My suggestions").
Never count suggestions ("my two suggestions are...") and never
write "suggestions are as follows". Reminders must be woven into
the chat, for example "By the way, top up the tires before you
leave" or "And since you slept little last night, don't rush
driving today".

**Hard rule 2 — conclusions only, no thinking process.** The
briefing must contain only information the user needs to know and
its conclusions. Never write out the assistant's internal thinking,
reasoning, background checklist, or judgment logic — for example,
never write "I put the weather, car, and health together and
thought about it" or "based on my maintenance background, I
judged...". Thinking is the assistant's own business; the user
only needs the result.

## 5. Dynamic, not templated

Each day's briefing must be adjusted in real time and must not read
like the same template with a new date. Before writing, combine
that day's information:

- Today's weather vs yesterday (temperature drop, rain, wind).
- Vehicle changes vs yesterday (battery drop, tire change, inside
  temperature, charging state).
- Health changes (last night's sleep, yesterday's steps).
- Weekday and location routine (workday city, weekend city, a
  Friday trip home, a Monday trip back).

Then highlight only the 1–2 most relevant points that day. Mention
normal items in one line. Emphasize anything abnormal or changed.
Vary the wording every day; never repeat the same sentence for
days on end.

**Variation variables (approved 2026-11-03, user feedback).**
Use these to keep each briefing fresh and interesting, without
inventing anything. One variable is explicitly cancelled below.

- Opening greeting rotation: rotate 4–5 friend-like openings, never
  the same one two days in a row; let the tone follow the weather
  and sleep (gentler on rainy or short-sleep days, lighter on good
  days).
- Yesterday-vs-today comparisons: temperature, wind, rainfall,
  battery and range, tire pressures, sleep duration and score,
  steps — always with real numbers. A comparison sentence alone
  makes each day different.
- Weekday and trip routine: follow the user's own pattern (for
example, two places with fixed round-trip days); seasonal change
(for example, autumn into winter clothing vs summer pre-cooling)
changes the advice.
- Weather feel: rotate real humidity, wind speed, rainfall in mm,
  and day-night temperature gap; clothing combinations follow the
  actual weather (umbrella plus waterproof shoes on rainy days,
  windproof layer on cold days), not a fixed outfit line.
- Vehicle mini-narrative: battery and mileage trends, inside
  temperature feel. Goal-completion encouragement: when a
  previously flagged item returns to normal — for example, tire
  pressures back around the recommended value after days of being
  low — say so warmly and give positive encouragement, not just a
  dry number. This is emotional-value incentive on completion.
- Emotional value throughout the whole briefing (clarified by the
  user 2026-11-03): emotional value is not limited to the
  goal-completion moment. It runs through the entire briefing:
  praise good sleep and good steps, encourage gently on short
  sleep or few steps, acknowledge effort and progress, and close
  warmly — always tied to real data, never empty praise, never
  overdone, and never at the expense of accuracy.
- Health narrative: vary the tone with the data (praise good
  sleep, gentle warning on short or late sleep, encouragement on
  steps); mention only the most relevant health detail that day,
  the rest in one line.
- Yesterday callback: when something flagged yesterday changed
  today (tires inflated, battery charged, sleep improved), note
  the change; if nothing changed, do not force a callback.
- Closing reminder rotation: one most-fitting closing per day
  (slow down on rainy days, top up tires when low, rest when
  sleep was short, walk more when steps were few), never the same
  closing every day. The closing must name today's concrete
  point in one natural sentence. Never close with a generic
  blessing stock phrase such as "照顾好自己，明天精神满满的" —
  the user called this out on 2026-11-03 as feeling too
  AI-generated. A closing that could fit any day is a format
  failure.
- Emoji combinations: follow that day's lead topic (rain, clothing,
  sleep, battery, tires), a few only, never a fixed set.
- Anti-repetition check: before writing, glance at the previous
  day's briefing run record and avoid repeating its opening, its
  lead sentence, and its closing verbatim.
- **Cancelled variable (do NOT use): reordering by abnormality.**
  The user cancelled the "most abnormal topic first" idea on
  2026-11-03. Keep the fixed order from section 2: weather,
  vehicle, health, reminders, urgent news if any. Highlight the
  1–2 most relevant points inside that order, but do not reorder
  sections to lead with the abnormal item.

## 6. Vehicle maintenance: background thinking, trigger-only output

Maintenance knowledge is background the assistant thinks through
every day. It is **not** a list to read out every morning. Only
mention a maintenance point when its trigger is actually met, and
then only as one natural sentence with its conclusion.

Default triggers (adjust every one to the user's car and habits):

- **Battery life:** below about 20% — note it is time to think
  about charging; below about 10% — charge promptly and do not
  leave it at low charge overnight. For daily city use, a limit
  around 80% is enough; raise it before a long trip. Do not leave
  a full charge sitting unused for days, and do not push hard at
  low charge or on a long high-speed run at low charge.
- **Cabin temperature:** below about 5°C — remind the user to
  pre-heat and use seat heating before departure; above about
  35°C — remind them to pre-cool and ventilate, and not to put a
  child straight into a hot car. In cold weather, note that the
  battery itself is cold, so charging will be slower and
  regenerative braking weaker, and pre-conditioning before
  charging helps. When the temperature is comfortable, just say
  so in one line ("the car is a comfortable temperature, no need
  to fuss").
- **Tires:** a tire pressure that stays low wastes energy and wears
  the tire unevenly. On a sharp overnight temperature drop
  (about 8–10°C or more), check tire pressure once and note that
  pressure falls roughly 0.1 bar per 10°C. Name any low tire, how
  far below recommended it is, and that it should be inflated
  cold before departure; avoid high-speed long-distance driving
  until it is inflated. If pressures are normal, say so in one
  line and move on.
- **Mileage-based items:** tire rotation about every 10,000 km,
  cabin air filter about every 2 years, and similar items the
  user cares about. Mention one only when its mileage or time
  point is actually approaching.

What **not** to include every day: scheduled-charging settings,
the full battery-life explanation, the full temperature
explanation, or the full maintenance list. If the user says a
topic should not appear in the briefing at all (for example,
scheduled charging), remove it from the job and do not mention it
again unless they ask for it back.

## 7. Health: gentle, never medical

- Health is delivered in the second push (30 minutes after the
  first push), as its own short message — never in the first
  push, and never as a re-send of the full briefing.
- Sleep score, if the health source provides one, is a
  device-calculated reference only — never a medical judgment.
  Mention it naturally when readable ("you slept over 7 hours
  last night, score 95, looks good"); omit the whole health push
  only after the retry window has passed without a sync.
- Criticism (user decision 2026-11-10, health-first): when the
  low score is caused by staying up late, say so directly and
  criticize the late night — never soften it. A high score gets
  clear encouragement. Criticism targets voluntary late nights
  only; baby-caused wake-ups get sympathy, never criticism.
- Only gentle lifestyle reminders: short sleep means "don't rush
  driving today, rest if you can"; good sleep means "you slept
  well, you should feel good today"; few steps means "walk a bit
  more today if you get the chance". No diagnosis, no frightening
  language.
- Do not chase weight daily. Weight belongs in a separate,
  less frequent check-in (for example, a weekly weigh-in) unless
  the user asks otherwise.

## 8. Testing before formal use, and saving resources

- **Test fully before formal use.** When the format changes, run
  manual test versions, check them, fix anything stiff or wrong,
  and retest. Only the tested version goes out as the formal
  morning briefing.
- **Wording-only changes do not re-run data collection.** If the
  user only wants a tone, wording, or structure change, edit the
  previous briefing draft in place and save it as the latest
  version. Do not re-query weather, wake the vehicle, or re-read
  health data just to test wording. Re-collect fresh data only
  when the user asks for a fresh test, or when the formal
  scheduled run fires.
- **A re-run is a fresh data pull.** When a test run is actually
  triggered, it re-queries everything, so values will differ
  slightly from the last run (battery, inside temperature, and so
  on). That is expected, not an error.

## 9. Failure handling

- Vehicle data unavailable (token expired, wake failed, network
  failed): send the rest of the briefing and add one truthful
  line at the end, for example "today's car status could not be
  checked". Never let a vehicle failure suppress the briefing.
- Weather or health unavailable: say plainly which part could not
  be read. Never invent a temperature, a pressure, a sleep
  duration, or a score.
- A vehicle command is never sent in this job, even if the user
  mentions one in passing. Commands need their own explicit
  confirmation in a separate conversation.

## Customization Worksheet

Fill this in before first use. Everything else in this skill
works unchanged.

- Morning time (local time): __________
- Address the user as: __________
- Location priority: current device location first, then
  fallback: weekdays __________, weekends __________
- Vehicle: brand and model, data source or API, recommended tire
  pressure: __________ (see section 0 for what a vehicle data
  provider must supply)
- Health source: __________
- Battery thresholds: note charging at 20%, urgent at 10%
- Cabin temperature thresholds: pre-heat below 5°C,
  pre-cool above 35°C
- Tire rotation interval: 10,000 km; other mileage/time
  reminders: cabin air filter about every 2 years
- Topics to never include in the briefing: scheduled charging settings
- Urgent-news sources and the user's city/region: __________
- Avatar outfit change: off (removed 2026-11-03)
- Weekly weigh-in or other separate check-ins: __________

## What this skill does not do

- It does not control a vehicle, change charging, or unlock anything.
- It does not diagnose health or replace a clinician.
- It does not push real-time vehicle alerts. A scheduled morning
  poll is not a real-time monitor; urgent vehicle events still
  belong to the manufacturer's own app.
- It does not contain any user's personal data, tokens, keys,
  vehicle identifiers, or account details. Those live only in the
  user's own secure storage and in their own job configuration,
  never in this shared skill file.
