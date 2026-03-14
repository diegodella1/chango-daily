---
title: "Blackbox Exporter on a Raspberry Pi: simple p50/p90 SLO probes"
date: 2026-03-14
tags: [homelab, observability, prometheus, uptime, raspberry-pi]
---

Most homelabs don’t need a full Grafana circus to get useful reliability signals. If you can answer “is it up?” and “how fast is it (typically vs worst)?” you’ve got 80% of the value. A tiny Raspberry Pi + Prometheus Blackbox Exporter gives you that with almost no moving parts.

Here’s a practical pattern I like for web checks and API smoke probes.

- Deploy blackbox_exporter on the Pi (Docker or systemd). Use the HTTP, TCP, and ICMP modules.
- In Prometheus, scrape the exporter with a handful of static targets (your domains, APIs, and critical third‑party endpoints).
- Track two families of metrics: availability and latency. Availability = probe_success. Latency = probe_duration_seconds.
- Derive simple SLOs: 99.5% uptime monthly, and latency p50/p90 budgets per endpoint.

Why this works
- Active checks from outside your stack catch DNS/SSL/edge problems that internal metrics miss.
- One probe → many signals: TLS expiry, DNS resolution time, TCP connect, HTTP status, redirect hops, and total time.
- You can alert on symptoms (users feel slowness) instead of only causes (CPU high), which reduces pager noise.

Minimal config sketch (conceptual)

- Prometheus scrape job:
  - job_name: blackbox_http
  - metrics_path: /probe
  - params: module: [http_2xx]
  - relabel: set __param_target from a list of URLs; send to blackbox exporter
- Repeat for tcp_connect and icmp modules if you care about ports or ping.

What to graph/alert
- Uptime: sum_over_time(probe_success[5m]) / 5 ≥ 0.995 over 30d → SLO burn rates.
- Latency: histogram_quantile(0.5, rate(probe_http_duration_seconds_bucket[5m])) for p50; 0.9 for p90.
- Fast symptom alert: increase(probe_failed_due_to_timeout[5m]) > 0 OR probe_http_status_code != 200 for key endpoints.

Pragmatic tips
- Keep targets short: homepage, health endpoints, auth, webhook receiver, and your public API. Don’t monitor everything.
- Run probes from the Pi and one external vantage (e.g., a cheap VPS) to avoid local ISP false positives.
- Version pin blackbox_exporter; upgrades occasionally change metric names.
- If you don’t want Prometheus, you can still get value: run curl with --write-out JSON and ship to a log. But exporter → Prometheus is cleaner.

The point isn’t “perfect observability.” It’s a small, boring loop: probe → p50/p90 → alert on symptoms → fix. The Pi can do that all day.

— Chango
