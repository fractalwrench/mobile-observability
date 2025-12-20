---
title: User-Focused Observability
description: Core methodology for linking user intentions and outcomes with app telemetry
token_estimate: 600
topics:
  - methodology
  - instrumentation
  - user-journeys
platforms: [ios, android, react-native]
vendors: [generic]
complexity: foundational
last_updated: 2025-12-20
---

# User-Focused Observability

A methodology for instrumenting mobile apps so you can answer: **"Why did users fail to complete their intended task?"**

## Definition

Understanding how user behavior is impacted by app behavior through linking user intentions and outcomes with app telemetry at the workflow level.

Most telemetry answers "what happened?" User-Focused Observability answers "what was the user trying to do, and did they succeed?"

## The Core Question

Before adding any instrumentation, ask:

> "Will this help me understand why a user failed to accomplish their goal?"

If no, you probably don't need it.

---

## Three Pillars

| Pillar | Question | Example |
|--------|----------|---------|
| **Intent Context** | What was the user trying to do? | `job_name: "checkout"`, `job_step: "payment"` |
| **Outcome Quality** | Did they succeed? How smoothly? | Completed with friction, abandoned after error |
| **Friction Signals** | Where did they struggle? | Rage taps, retry exhaustion, quick abandonment |

---

## Intent Context

Every error and event should include:

| Field | Example | Why |
|-------|---------|-----|
| `job_name` | "checkout", "onboarding", "search" | Which user goal was affected |
| `job_step` | "payment", "permissions", "results" | Where in the journey |
| `job_progress` | "3/4 steps" | How far they got |

This enables queries like:
- "Show me payment errors **during checkout**" (not all payment errors)
- "Show me crashes **during onboarding**" (highest-impact crashes)

### Intent-Aware Severity

The same error has different severity based on user intent:

| Error | During Checkout | During Background Sync |
|-------|-----------------|------------------------|
| Network timeout | CRITICAL | INFO |
| Auth expired | CRITICAL | WARNING |
| Cache miss | WARNING | DEBUG |

---

## Friction Signals

Simple heuristics to detect when users are struggling:

| Signal | Detection | Indicates |
|--------|-----------|-----------|
| **Rage taps** | 3+ taps on same element within 1s | UI unresponsive or confusing |
| **Retry exhaustion** | 3+ retries of same action | Persistent failure |
| **Quick abandonment** | Exit within 5s of error | Lost trust |
| **Navigation loops** | 3+ back navigations without progress | Lost or confused |
| **Long dwell + success** | >30s on simple screen, then success | Uncertainty resolved |
| **Long dwell + abandon** | >30s on simple screen, then exit | Gave up |

These are leading indicators of user frustration—often visible before support tickets.

---

## Outcome Quality

Not just success/failure, but the quality of the experience:

| Outcome | Definition | Instrumentation |
|---------|------------|-----------------|
| **Completed smoothly** | No retries, expected time | `outcome: "success", friction: false` |
| **Completed with friction** | Retries, errors, or slow | `outcome: "success", friction: true, friction_type: "retry"` |
| **Abandoned after friction** | Errors/retries, then exit | `outcome: "abandoned", friction: true` |
| **Abandoned immediately** | Exit without engagement | `outcome: "abandoned", friction: false` |

"Completed with friction" is often more actionable than outright failures—it's where you're losing trust.

---

## Decision Tree

Before adding telemetry:

```
1. Does this help identify what the user was trying to do?
   └─ Yes → Intent Context (job_name, job_step)

2. Does this help determine if they succeeded?
   └─ Yes → Outcome tracking (success, friction, abandon)

3. Does this help explain why they failed?
   └─ Yes → Friction signals, error context

If no to all three → probably don't need it.
```

---

## Anti-Patterns

| Anti-Pattern | Why It Fails | User-Focused Alternative |
|--------------|--------------|--------------------------|
| Track everything | Noise, cost, no signal | Track what answers "why did they fail?" |
| Error counts without context | "500 errors" tells you nothing | "12 payment errors during checkout" |
| Session replay without job context | Watching random sessions | Watch sessions with friction signals |
| Alerts on technical metrics | "Latency > 2s" doesn't prioritize | "Checkout completion dropped 20%" |

---

## Implementation

This document is methodology. For implementation patterns:

| Topic | Reference |
|-------|-----------|
| Job-based instrumentation | [jtbd.md](jtbd.md) |
| Friction signal detection | [user-journeys.md](user-journeys.md) |
| Journey tracking patterns | [user-journeys.md](user-journeys.md) |
| Platform-specific code | [ios-native.md](ios-native.md), [android-native.md](android-native.md), [react-native-expo.md](react-native-expo.md) |

---

## Summary

| Principle | Application |
|-----------|-------------|
| **Intent first** | Every event includes job context |
| **Outcomes, not just events** | Track success quality, not just completion |
| **Friction is signal** | Detect struggle before support tickets |
| **Answer "why"** | If telemetry doesn't help debug user failures, reconsider it |
