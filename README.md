# Telegram Life Assistant

A personal AI life assistant on Telegram, built on Google Apps Script and Google Sheets. Runs my entire daily schedule — diet, gym, work blocks, UPSC study, supplements, sleep prep — across four distinct day-types, with automated reminders, response tracking, and a nightly completion summary. Total infrastructure cost: **₹0/month**, runs on Google’s free tier.

## Why I built this

I was juggling seven concurrent priorities — full-time PM role at Axis Bank, UPSC CSE preparation, a side-business (orc.a, a paper company), modernizing the family interior business, gym and diet, skincare and grooming, and reading. Context-switching between these without losing momentum was the actual product problem.

Existing solutions sat at two unhelpful extremes. Calendar apps are too generic — they just buzz at you, no concept of which goal a reminder is serving, no logging, no review. Full productivity systems (Notion, Sunsama, Reclaim) are too heavy — they themselves become chores to maintain.

I wanted a single zero-friction surface — Telegram, which I already check 50× a day — to ask me one thing at a time, in the right order, on the right day, and log whether I actually did it.

## How it works

Every minute, two Google Apps Script triggers run in parallel:

- **`checkSchedule`** — looks at the current time and current day-type, finds any scheduled reminder due to fire, and pushes it to my Telegram chat with three inline buttons: `✅ Done` / `⏭ Skip` / `⏰ +30 min`.
- **`pollTelegram`** — polls the Telegram API for any button presses or text replies, parses the response, and logs it to Google Sheets with timestamp and reminder ID.

At 11 PM, a separate trigger runs `sendDailySummary`: it pulls all responses from the day, calculates a completion percentage, and sends a single Telegram message summarizing what got done, what got skipped, and the score for the day.

> *Architecture diagram to be added — draw.io export coming.*

## Four day-type variants

The schedule isn’t a single fixed template — different days have genuinely different structures, and forcing one schedule onto all of them is why most personal scheduling systems get abandoned within a week.

|Day type            |Defining feature                                             |
|--------------------|-------------------------------------------------------------|
|**WFH (Mon–Wed)**   |Gym block at 1:00 PM, deep-work blocks before and after lunch|
|**Office (Thu–Fri)**|Walk-home as cardio, no separate gym session                 |
|**Saturday**        |Two UPSC study blocks, life admin, balanced training         |
|**Sunday**          |Morning gym before noon, weekly reset and review             |

Each variant has 20–30 reminders covering wake-up, hydration, supplements (vitamin D, B12, omega-3), three meals plus an evening snack, work blocks, UPSC study windows, gym, skincare stack, and sleep prep.

## The most interesting product decision: polling over webhooks

The initial design used webhooks — Telegram pushes a notification to a Google Apps Script web app URL whenever a button is pressed, the script processes it and replies. Clean architecture, low latency, the textbook solution.

It didn’t work. Apps Script’s web app deployment returns a `302` redirect on the initial request, and Telegram’s bot API doesn’t follow redirects on webhook callbacks. After a few hours of debugging, I switched to a **polling architecture**: the script asks Telegram every 60 seconds for any new messages or button presses, processes whatever’s there, and writes to the log.

This trades ~30 seconds of latency on average for two things: it works reliably inside Google’s free tier with zero infra, and it removes the entire class of “is my webhook endpoint reachable” failure modes. For a personal tool where 30-second latency is invisible, this was the right call. The lesson — pick the architecture that matches the actual reliability and cost constraints, not the one that sounds more elegant on paper — is the most generally useful thing I took away from this project.

## Stack

- **Backend:** Google Apps Script (free, always-on, 6-minute execution limit per run)
- **Database:** Google Sheets — one tab per log type (reminders fired, button responses, daily summaries)
- **Interface:** Telegram Bot API via [@hriday_life_bot](https://t.me/hriday_life_bot)
- **Triggers:** Two 1-minute time-based triggers + one daily 11 PM trigger
- **LLM (planned):** Claude API for free-text queries beyond the fixed schedule

## What it’s actually done for me

> **TODO — replace this block with one specific example.** A real day where the bot kept you on track, or a rolling adherence number (“70%+ task completion across the last 30 days”), or the moment you realised it was working. Two or three sentences, concrete.

## Roadmap

- **Weekly review summary** every Sunday evening: rolling 7-day completion percentage, longest streak per category, and the single most-skipped reminder of the week.
- **LLM integration.** Conversational queries from Telegram: *“what should I be doing right now?”*, *“log my dinner”*, *“move tonight’s gym to 9 PM”* — routed through Claude with the schedule and recent log as context.
- **Multi-channel.** A read-only WhatsApp Business mirror so my parents can see the daily summary too.

## A note on code visibility

The live script contains credentials specific to my Telegram bot (token, chat ID) and the underlying Google Sheets (spreadsheet ID), so the production version isn’t published here. Happy to share a sanitized template or walk through specific parts of the architecture in a conversation.

-----

**Built by [Hriday Mehta](https://www.linkedin.com/in/hridaymehta56)** — Product Manager working on cross-border payments and US equity investments at Axis Bank. Reach out: mehtahriday5@gmail.com.
