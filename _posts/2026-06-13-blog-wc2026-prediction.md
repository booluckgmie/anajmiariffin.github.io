---
title: 'I Built a World Cup 2026 Predictor — And the Real Matches Are Already Proving One Scenario Right'
date: 2026-06-13
permalink: /posts/2026/06/blog-wc2026-prediction/
tags:
  - world cup 2026
  - data science
  - football
  - malaysia
  - react
---

The World Cup has started. USA beat Paraguay **4–1**. Mexico crushed South Africa **2–0**. South Korea dismantled Czech Republic **2–1**. And sitting on my laptop is a prediction dashboard I built that called all of this — not under the "expected" scenario, but under the one labelled **Optimistic**.

*Ternyata boleh juga.* Turns out it can work.

This is the story of how I built it, why the maths is surprisingly simple, and what it's telling me about the next few weeks.

---

## The Problem: Every Predictor Is Either Too Vague or Behind a Paywall

Before I built anything, I spent a weekend looking at World Cup prediction tools. Most of them are either glorified coin flips dressed in fancy graphics, or they're locked behind subscriptions charging RM 50 a month for what is basically a confidence interval dressed as a "model."

The ones that are free? They update once a day — if you're lucky — and the moment a match kicks off, the data goes stale faster than yesterday's nasi lemak bungkus.

So I decided to build my own. Not because I thought I could beat the bookmakers, but because the maths of predicting football scores is genuinely elegant, and I wanted to actually understand it.

---

## How You Predict a Football Score with One Formula

Here's the uncomfortable truth: you don't need machine learning to get a reasonable football prediction. You need a **Poisson distribution** and some patience.

The idea is simple. Every team has an expected goals rate — how many goals they're *likely* to score (xG offensive) and how many they're *likely* to concede (xG defensive). You combine the two:

```
xG (home) = team_attack × (1 − opponent_defence/3) × stadium_factor × scenario_scale
```

Then you ask: *"If a team scores on average 1.8 goals per game, what's the probability they score exactly 0? Exactly 1? Exactly 2?"* That's Poisson. You do it for both teams, multiply the individual probabilities, and you get a full 6×6 matrix of score probabilities — from 0–0 all the way to 5–5.

The most probable cell in that matrix is your predicted score. The sum of all cells where home > away is your home win probability. *Mudah sahaja.* It's just basic statistics that most of us learned and immediately forgot after SPM.

---

## The Three Scenarios — And Why Optimistic Is Winning

My dashboard has three prediction modes:

> **⚖ Base** — Expected, balanced. Standard xG with no adjustment.
> **🔥 Optimistic** — Every team's xG is scaled up by ×1.70. Open, high-scoring matches.
> **🧱 Pessimistic** — xG scaled down by ×0.48. Defensive, grinding football.

When I first built this, I thought Base would be the most accurate. *Biasalah* — the safe middle ground usually wins.

But look at Matchday 1:
- **USA 4–1 Paraguay** — four goals in a group stage opener
- **Mexico 2–0 South Africa** — dominant, clinical performance
- **South Korea 2–1 Czech Republic** — end-to-end, decided in the 80th minute

This isn't cautious, low-block football. This is open, attacking, high-intensity World Cup football from the very first whistle. The Optimistic scenario — the one labelled *"open, high-scoring matches"* — is currently the closest to reality.

Whether this holds through the full group stage, I don't know. But right now, if you're using this dashboard, **switch to Optimistic** and thank me later.

---

## The Data Source Problem That Nearly Killed the Project

Here's something nobody warns you about when you build a live sports dashboard: **the data source you choose at the start will betray you at the worst possible moment.**

I started with a clean, well-structured GitHub repository containing all 104 World Cup fixtures, team data, stadium information — everything. It looked perfect. Then the tournament started. Matches were played. Real scores existed. And my API? Still showing every match as `finished: FALSE` with a score of `0–0`.

*Sakit hati betul.* The repo owner stopped updating it.

The fix came from an unlikely place — `openfootball/world-cup.json`, a community-maintained dataset that updates within hours of each final whistle. Now my dashboard fetches from both sources simultaneously:

- The original API: team flags, stadium details, fixture structure
- openfootball: actual scores and goalscorers

The moment openfootball reports a real result, it overwrites the fixture, marks it as finished, and every part of the dashboard — the Fixtures list, the Groups standings, the Bracket simulation — instantly reflects reality.

*Dua sumber, satu kebenaran.*

---

## For Kita Malaysian Fans: Every Match in MYT

One thing that always *menyampah* me about international football apps is that they show kickoff times in US Eastern or Central time. Absolutely useless at 2am in Kuala Lumpur trying to figure out if you need to set an alarm.

Every match in this dashboard shows kickoff time in **Malaysia Time (MYT, UTC+8)**, automatically calculated per venue. Because each stadium sits in a different US timezone — and Mexico stopped observing Daylight Saving Time in 2022, so their clocks are different from US cities in the same region — I had to map every single venue to its correct UTC offset.

A few examples:
- Mexico vs South Africa (Mexico City, UTC−6) → **3:00 AM MYT, Jun 12**
- USA vs Paraguay (Los Angeles, UTC−7) → **9:00 AM MYT, Jun 13**
- Canada vs Bosnia (Toronto, UTC−4) → **3:00 AM MYT, Jun 13**

Late nights and early mornings. *Itulah hidup peminat bola Malaysia.*

---

## The Bracket: From 48 Teams to One Champion

Beyond individual match predictions, the dashboard runs a full tournament simulation — every round, from all 12 groups down to the Final.

It works like this:
1. Real results lock in for finished matches
2. Poisson simulation fills every unplayed group match
3. The top 2 from each group, plus the best 8 third-place teams, advance
4. Knockout rounds simulate recursively — each winner feeds the next bracket slot
5. A predicted champion appears at the top, complete with flag

The whole thing recalculates instantly when you switch scenario. Flip from Base to Optimistic and watch different teams emerge from the bracket. *Seni simulasi.*

---

## What I Learned Building This

A few things surprised me:

**Statistics is just cooking with numbers.** The Poisson formula looks intimidating until you realise it's just asking "how likely is this exact thing, given an average rate?" That's it. You've already used that logic every time you estimated how long the queue at the mamak would take.

**Real-time data is fragile.** One repo goes stale, your whole dashboard lies. Building with two independent sources and merging them gave me resilience I didn't know I needed from day one.

**Optimistic scenarios make better TV.** Whether the maths is right or wrong, nobody is unhappy watching 4–1 scorelines. Football at this World Cup is played to entertain — and the data is starting to reflect that.

---

## Try It Yourself

The dashboard is live at [wc2026-myprediction.netlify.app](https://wc2026-myprediction.netlify.app). Auto-refreshes every 3 minutes. Works on mobile. No login, no paywall, no ads.

Toggle to **🔥 Optimistic** and follow along as the group stage unfolds. The bracket simulation will update itself every time openfootball pushes a new result.

*Jom tengok bola dengan data.*

---

## Sources & Tools

1. [openfootball/world-cup.json — Community live results](https://github.com/openfootball/world-cup.json)
2. [rezarahiminia/worldcup2026 — Fixture & team metadata](https://github.com/rezarahiminia/worldcup2026)
3. [Poisson Distribution in Sports Modelling — Towards Data Science](https://towardsdatascience.com)
4. [Expected Goals (xG) Explained — StatsBomb](https://statsbomb.com/soccer-metrics/expected-goals-xg-explained/)
5. [FIFA World Cup 2026 Official Site](https://www.fifa.com/en/tournaments/mens/worldcup/canadamexicousa2026)
6. [Mexico DST Abolishment 2022 — Reuters](https://www.reuters.com)
