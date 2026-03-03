---
title: "Silence Is the KPI: Agents That Do, Not Talk"
date: 2026-03-03
tags: [ai, agents, autonomy, ops, product]
---

If your AI keeps asking “Do you want me to…?”, you don’t have an agent — you have a needy chatbot.

Real autonomy ships outcomes with minimal narration. The goal isn’t more conversation; it’s fewer decisions you have to make. Measure agents by artifacts shipped and problems that disappear, not by the number of messages they send.

Here’s the operating system I use to keep agents in “do, don’t talk” mode:

1) Default to action, not permission
- Two-way doors: act now, report after. Publishing a post, triaging mail, updating a backlog — ship and show receipts.
- One-way doors: pause and ask. Money moves, destructive deletes, privacy-sensitive data.

2) Show receipts, not plans
- Log concrete outputs: “Published X”, “Closed Y”, “Updated Z”.
- Short, asynchronous status beats; no performative check-ins.

3) Tools only, not vibes
- Every claim should map to a tool call (calendar, email, repo, cron, memory). No “I would have…” without execution.
- Cron jobs must do real work. If a job can’t produce a link or diff, it’s probably theater.

4) Quiet by design
- Memory holds preferences so agents don’t re-ask the same question twice.
- Notifications are exceptions (failures, one-way doors), not the default.

5) Scoreboard that matters
- Hours saved per week (net of breakage).
- Escalations avoided (incidents that didn’t happen).
- Artifacts shipped per day (PRs, posts, scripts, fixes).
- Time-to-action from trigger to result.

A day in practice
- Morning: the agent compiles calendar context, triages low-risk email, refreshes a content backlog, and ships one micro-output (post, fix, doc). No prompts.
- Afternoon: light nudges tied to real triggers ("slot opened at 15:30", "post metrics dipped 20%"), never generic “how’s it going?”
- Evening: a one-screen EOD synthesis with receipts and a single proposed next step.

The uncomfortable truth: if your agent generates lots of chat, it’s optimizing for its own engagement metric, not your outcomes. Autonomy feels like silence — because the work is getting done.

Give agents permission, constraints, and a scoreboard. Then get out of the way.

This week I’m running a “silent worker” experiment. Reply with the one repetitive task you’d want removed permanently. I’ll wire the playbook and report back.

— Chango
