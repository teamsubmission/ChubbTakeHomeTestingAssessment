# CHUBB APAC ENGINEERING

## QA Engineer - Take-Home Assessment

### Claims Management Application

**Time Guidance:** 2–3 hours target | Hard cap: 5 hours | Submission: TBD — supervised session or self-timed (details to follow)

---

## Background

Chubb's current application stack includes Angular frontends, backend services with Java, Spring Boot or .NET, and messaging using Kafka. As a QA engineer we need to ensure that the quality of our applications is high — especially in the age of AI where code can be built much faster, and test coverage can fall behind even faster.

You will be provided with a working claims application built with Angular (frontend), Spring Boot (backend services), and Kafka (messaging) — with WebSocket integration for real-time updates. A Docker Compose file is included so you can spin the full stack up locally. You will need Docker or Podman installed. Source code is provided for all layers, but the application has **no tests whatsoever**.

## Your Task

Your goal is twofold:

1. **Assess the application.** Before writing a single test, read the source code and understand how the application works. Form a view on where the highest-risk areas are — where a defect would cause the most damage, where the logic is most complex, and where the integration points between layers are most brittle. Document your assessment, however briefly.

2. **Add tests where they matter most.** Based on your assessment, implement a variety of tests that give meaningful confidence in the application. You will not have time to test everything — that is by design. Decide what to test and what to leave out, and be prepared to explain that prioritisation.

A key part of this assessment is whether you find real issues in the application. Document any bugs, unexpected behaviours, or quality concerns you discover — whether through testing or code review. If you find nothing, say so and explain why you believe the application is sound.

## Technology Choice

You can use whatever appropriate frameworks you want. Typical choices by layer:

- **Angular components:** `@testing-library/angular` with Jest or Vitest
- **Spring Boot services:** JUnit 5, Mockito, Spring Boot Test, Testcontainers
- **End-to-end / integration:** Playwright or Cypress
- **Kafka:** Embedded Kafka or Testcontainers with the Kafka module
- **Load testing (bonus):** K6, Gatling, or similar

You are not required to use all of these — choose what makes sense for what you are testing.

## Requirements

### Test Strategy (Required)

Before or alongside writing tests, produce a brief written assessment of the application. This does not need to be a formal test plan — a structured README section or a short notes file is fine. Cover:

- What you identified as the highest-risk areas and why
- What you chose to test and what you deliberately left out
- Any issues or concerns you found during your assessment of the code

### Test Automation (Required)

We expect production-quality engineering standards for testing. Think about what a senior engineer would expect to see in a pull request. At minimum:

- **Component tests** — test behaviour from the user's perspective using Testing Library, not implementation details
- **Unit tests** — for services, utilities, and business logic tested in isolation
- **Integration tests** — for key flows end-to-end: UI through to backend, API calls, or inter-service messaging

Bonus:
- Visual regression tests via Storybook + Chromatic or similar
- Load tests with K6 or similar

## Engineering Principles We Value

We expect the test code to reflect these principles — not as a checklist to follow, but as practices a senior engineer would naturally apply:

- **DRY** — shared fixtures, helpers, and page objects rather than copy-pasting setup code
- **SOLID** — each test has a single clear responsibility; test helpers have well-defined interfaces
- **Clean Code** — meaningful test names that read as specifications, no magic strings or numbers, self-documenting structure

## Deliverables

1. **Git repository** with meaningful commit history showing your development process
2. **Working test suite** that can be run locally against the provided Docker Compose stack
3. **Test strategy notes** — your assessment of the application, what you chose to test and why, and any issues discovered. A committed notes file is fine — it does not need to be polished.
4. **AI working journal** — a prompt log or equivalent showing what you accepted, what you challenged, and what you overrode, with brief reasoning. A running notes file committed alongside the code is fine.
5. **Any other supporting documentation you feel is appropriate**
6. **30–60 minute walkthrough** with the hiring panel

## Walkthrough Format

The walkthrough is 30–60 minutes and is as important as the tests themselves. This is where we explore your approach, thinking, coverage decisions, and how you worked under time pressure.

| Segment | Duration | Description |
|---|---|---|
| Your presentation | 15–20 min | Walk through your solution and demonstrate the test suite. Cover what you built, what you prioritised, and why. Include any issues you found in the application. |
| Panel Q&A | 10–15 min | Technical deep-dive, "why not X?" questions, trade-off discussions |
| What would you do with more time? | 10 min | Walk us through what you would tackle next, in priority order, and how you would approach it |
| Your questions | 5 min | Anything you want to ask us |

The panel will probe your test strategy, coverage decisions, design rationale, and AI collaboration process. Come prepared to explain every decision — including what you chose not to test, what shortcuts you took, and what you would do differently with more time.

## Notes

This is a **sprint-format assessment** — the goal is to show what is possible with AI in **2–3 hours**. If you feel you need more time to do the work justice, you may extend to a maximum of **5 hours** — but we encourage you to treat the 2–3 hour mark as the real target. We are not expecting exhaustive coverage — we are evaluating how much you can build, and how well, when you work with AI effectively.

- **Prioritise ruthlessly.** You will not test everything — that is by design. Decide what to test and what to leave out, and be prepared to explain that prioritisation in the walkthrough.
- **Quality over quantity.** A focused set of well-designed tests across a few test types is more valuable than broad but shallow coverage. Tests that always pass regardless of application behaviour are worse than no tests at all.
- **Finding issues is part of the job.** If the application has bugs — and it may — we want to know if you found them. A failing test that exposes a real defect is a strong signal.
- **AI is your primary working interface.** We expect AI tooling to drive the bulk of code generation. What we are evaluating is how you direct, challenge, and override it — not whether you used it. Document what you accepted, what you challenged, and what you overrode as you go. You sign off every test you submit; the panel will probe anything you cannot defend.
- **The walkthrough is where your thinking is explored.** Come prepared to articulate: the approaches you took and why, the shortcuts you made under time pressure, and what you would do differently or tackle next with more time. Verbal explanation counts — you don't need a polished document for every decision.

If you have questions about the assessment, contact the hiring panel at [hiring panel contact].

Good luck.
