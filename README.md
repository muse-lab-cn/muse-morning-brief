# Muse Morning Brief / 每日早报

A daily morning briefing skill: one short, warm, first-person message
that reads like a good friend checking in — not a report, not a
checklist. Weather, vehicle status, health, and personalized
reminders, plus at most one urgent news item, only when one truly
matters.

Part of [Muse 研究所](https://github.com/muse-lab-cn). Previously an
optional skill inside [MuseDrive](https://github.com/muse-lab-cn/MuseDrive);
spun out as a standalone project in November 2026.

中文说明见 [README.zh-CN.md](README.zh-CN.md)。

## What it does

Every morning, in two pushes:

1. **Push 1 — weather + vehicle**, at a time you choose. Today's
   weather and clothing advice, read-only vehicle status (battery,
   range, charging, tire pressures vs. recommended, lock state,
   inside/outside temperature), woven reminders, and urgent news
   only if any.
2. **Push 2 — health**, 30 minutes later. Last night's sleep
   (duration, bed/wake times, sleep score), yesterday's steps and
   resting heart rate, plus one gentle reminder. If sleep has not
   synced yet, it retries every 5 minutes for up to an hour, then
   skips that day silently. It never invents data.

The tone is a good friend, not a report: no bold section headings,
no counted suggestion lists, conclusions only. Each day is adjusted
in real time — weather vs. yesterday, vehicle vs. yesterday, health
vs. yesterday — highlighting the 1–2 most relevant points, with a
freshness mechanism so it never reads like the same template with a
new date.

## Vehicle data providers

The vehicle part is a **data interface**, not a brand. Any car or EV
whose status can be read can be adapted, as long as its provider can
supply, read-only and on demand: battery % and range, charging state
and limit, all four tire pressures plus the recommended value, lock
state, inside/outside temperature, and (optionally) total mileage
read live.

Rules for every provider: read-only always, no vehicle commands;
wake at most once per briefing and only with the user's explicit
authorization; no continuous tracking.

**Tested provider:** Tesla, via the Tesla Fleet API — the only
provider tested end-to-end so far (see the sibling project
[MuseDrive](https://github.com/muse-lab-cn/MuseDrive)). Other brands
and EVs can be adapted by implementing the same interface; see
section 0 of `SKILL.md` for the field table and adaptation steps.

## Health sources

Any health source the user has authorized and that syncs sleep and
activity data — for example Apple Health. The skill judges sleep
against the user's age band (filled in on the Customization
Worksheet), never as a medical diagnosis. A second-opinion check
with another AI model is optional, never required.

## How to use

1. Read `SKILL.md` (English) or `SKILL.zh-CN.md` (Chinese).
2. Fill in the Customization Worksheet at the end with your own
   morning time, address, places, vehicle and data source, health
   source, thresholds, and urgent-news sources.
3. Give the filled-in skill to your AI assistant and follow its
   instructions. Do not paste private keys, tokens, secrets, or
   vehicle identifiers into a skill file or a prompt — those live
   only in your own secure storage and job configuration.

## What this skill does not do

- It does not control a vehicle, change charging, or unlock anything.
- It does not diagnose health or replace a clinician.
- It does not push real-time vehicle alerts. A scheduled morning
  poll is not a real-time monitor; urgent vehicle events still
  belong to the manufacturer's own app.
- It does not wake a sleeping vehicle without the user's explicit
  authorization, and never sends a vehicle command.
- It contains no user's personal data, tokens, keys, vehicle
  identifiers, or account details.

## Security

See [SECURITY.md](SECURITY.md).

## License

MIT License. See `LICENSE`.
