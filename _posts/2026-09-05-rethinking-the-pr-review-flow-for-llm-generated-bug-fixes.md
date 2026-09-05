---
layout: post
title: Rethinking the PR Review Flow for LLM-Generated Bug Fixes
---
> _“A wizard is never late, nor is he early; he arrives precisely when he means to” ~ Olórin aka Mithrandir aka Gandalf._  

In answer to why I took 2 years to write a new blog post...

# Rethinking the PR Review Flow for LLM-Generated Bug Fixes

As a security engineer, some percentage of my job involves finding bugs in code I don't own. 
LLMs have made that part faster. What's coming next is LLMs proposing the fixes too, and that's where things get complicated.
When a security team finds a vulnerability, the usual path is to file a report and wait for the owning team to act on it. 
That team has their own priorities, their own deadlines, and often limited context on why the fix matters. 
If a security-finding comes with an LLM-generated fix attached, the calculus changes: the developer doesn't have to write anything, they just have to review and merge. That's a much lower ask.
 
But it only works if the developer can trust the fix won't break their service. 
Most developers are not security specialists. They're not positioned to deeply audit an LLM-generated patch for a vulnerability class they've never thought about. 
What they can evaluate is much simpler: **does this change the observable behaviour of my service?** If the answer is no, and the fix was proposed by a tool the security team vouches for, they'll merge it.
 
If that trust breaks once, the whole model collapses. 
A bad auto-fix that causes an incident means the next fix goes to the bottom of the backlog, reviewed with suspicion, or ignored entirely. The security team is back to filing reports and waiting.
 
So the end goal, from a security engineering perspective, is a PR flow where the developer can be confident the fix is functionally inert until validated. Not confident because they read the diff carefully. Confident because the system gave them evidence.
 
That's what this post is about.

---

LLM-based coding agents are becoming a routine part of the development workflow: generating fixes, refactoring critical paths, and opening pull requests autonomously. 
But there's a problem I have not seen fully addressed yet. The PR review process was designed for human-authored code, and it is not a good fit for machine-authored changes.

When a human writes a bug fix, a reviewer can interrogate intent. They can ask "why did you do it this way?" and get a meaningful answer. 
The author has context, has thought through edge cases, and has some skin in the game. 
When an LLM generates a fix, none of that is true. The diff can look clean and plausible while being subtly wrong in ways that only manifest under specific runtime conditions: the kinds of things that unit tests and static analysis miss.

So what should the PR review flow look like when the author is an LLM?

## Prior Art: GitHub Scientist

Before proposing anything new, it's worth acknowledging that the core pattern here isn't entirely novel. GitHub's [Scientist library](https://github.com/github/scientist) is a Ruby library developed by Jesse Toth that has been quietly solving a related problem since 2016.

The Scientist pattern works like this: you wrap a code change so that both the old path (the *control*) and the new path (the *candidate*) execute on every request. The control's result is always returned to the caller. The candidate runs in parallel, its result is compared to the control's, and any discrepancy is logged without ever affecting the user. When you're confident the candidate matches the control closely enough, you cut over.

There are ports of Scientist to most major languages: [Scientist.NET for C#](https://github.com/github/Scientist.net), [Tzientist for Node.js/TypeScript](https://github.com/TrueWill/tzientist), and others. The pattern is also known as the **Parallel Run pattern**.

What nobody has proposed is letting an LLM generate this scaffolding automatically, as part of a structured PR workflow. That's the gap this post is about.

## The Problem with Today's LLM PR Flow

When a coding agent like Claude Code or Codex generates a bug fix, a flow I've observed in enterprises is:

1. LLM generates a fix and pushes a branch
2. LLM opens a PR (sometimes policy dictates only a human can create these)
3. Human reviews the diff
4. Human approves (or doesn't)
5. Change is merged and deployed

The human reviewer is being asked to do something very difficult: predict, from reading a diff alone, whether the new code will behave correctly in production under real traffic. This is a confidence problem. The diff might look right. The tests might pass. 
But critical code paths often have emergent behaviour under production conditions: race conditions, unusual input distributions, third-party timing dependencies. No amount of static review catches those.

We're essentially asking reviewers to trust a black box by staring at its output. 
Additionally, if this change introduces a service outage, there is no LLM to engage to help the reviewer fix the issue, the reviewer suddenly has to focus on fixing this issue and takes the blame for it.

## A Better Flow: LLM-Orchestrated Shadow Execution
 
Here's the flow I think makes more sense for LLM-generated changes to non-trivial code paths. The key insight is to separate *readying the data collection* from *making the change*, and to let the LLM manage the scaffolding lifecycle rather than requiring humans to do it.
 
### Step 1: LLM creates a feature flag
 
When generating a fix, the LLM doesn't just produce a diff. It also creates a new feature flag in the production feature flag system (LaunchDarkly, Flagsmith, Harness, etc.) keyed to this specific change.
 
### Step 2: LLM wraps the change in control/candidate flow
 
Rather than a simple diff that replaces the old code, the LLM wraps the change in a Scientist-style experiment:
 
```python
def process_joekir_webhook(event):
    with experiment("llm-fix-joekir-webhook-2026-09-05"):
        control:
            return legacy_webhook_handler(event)
        candidate:
            # Ideally the LLM-generated fix runs in a separate thread/process
            return new_webhook_handler(event)
    # Always returns the control result during the experiment
```
 
The candidate runs in parallel (in a separate thread or process depending on the risk profile), its result is compared to the control, and discrepancies are logged as structured A/B observations. The control result is always what the system acts on.
 
### Step 3: LLM opens the PR and says exactly what it is
 
This is where the PR review flow diverges meaningfully from today's model. The LLM opens a PR with a description that makes the nature of the change explicit:
 
> "This PR does not change any production behaviour. It wraps the existing joekir webhook handler in an experiment harness that will run the proposed fix in a shadow thread and log outcome differences for observation. No user will be affected. Please review the experiment scaffolding and approve data collection to begin."
 
The reviewer's job is now much more tractable. They are not being asked to predict runtime behaviour from a diff. They are being asked to sanity-check an experiment setup. The blast radius of a mistake is zero: if the candidate panics, the control result is returned. If the candidate produces a different result, it's logged but discarded.
 
### Step 4: Reviewer approves and the change is deployed
 
The PR is reviewed and deployed. From a user perspective, nothing has changed. Under the hood, the candidate code path is now running in shadow on every request to the affected code path and logging its outcomes.
 
### Step 5: LLM proposes an observation window
 
As part of the PR, the LLM proposes an appropriate observation window: a duration after which there will be enough production traffic to draw a meaningful conclusion. For a high-traffic webhook handler, that might be 48 hours. For a rarely-invoked batch job, it might be two weeks. The LLM can base this on the invocation frequency it can infer from logs or codebase context.
 
The human reviewer approves the proposed window (or adjusts it) as part of the PR review. A timer is set.
 
### Step 6: LLM notifies with data when the timer fires
 
When the observation window closes, a second LLM task fires. It reads the experiment logs, summarises the A/B comparison (match rate, error rates, latency differences, notable discrepancies), and opens a notification wherever the team works:
 
> "Experiment `llm-fix-joekir-webhook-2026-09-05` complete. 99.97% match rate across 412,000 observations. Candidate was 12ms faster on average. No errors. Recommending cutover."
 
The human makes the go/no-go call with actual data, not just code review intuition.
 
### Step 7: Cutover
 
The feature flag is flipped. The candidate becomes the production path. The control is disabled. At this point the real code change has happened, validated by production data.
 
### Step 8: LLM opens a cleanup PR
 
This is the part that already exists in isolation. Harness FME, LaunchDarkly, and Warp's Oz Agent all have tooling for automated feature flag cleanup. But in this flow, it's the final act of the same LLM-orchestrated lifecycle. The LLM opens a PR that removes the experiment scaffolding, inlines the candidate as the new control path, and deletes the feature flag definition.
 
The human reviews a much simpler diff and merges.
 
## What Makes This Different from Shadow Mode for LLMs
 
It's worth distinguishing this proposal from "shadow mode" as it's discussed in the LLMOps literature. When [Ramp runs their financial automation agents in shadow mode](https://www.zenml.io/blog/what-1200-production-deployments-reveal-about-llmops-in-2025), they're running an *LLM agent* in shadow: mirroring real transactions to see what the agent *would* have done before trusting it to act. Similarly, [shadow testing for LLM model upgrades](https://tianpan.co/blog/2026-04-09-llm-gradual-rollout-shadow-canary-ab-testing) involves running a candidate model alongside a production model to compare outputs.
 
This proposal is different. The LLM is the *author and lifecycle manager* of a shadow experiment for *regular code*, not itself the thing being shadowed. The LLM generates and maintains the experiment scaffolding; humans provide the governance checkpoints.
 
## The Review Bottleneck Problem
 
The standard response has been to add LLM reviewers to the PR process: CodeRabbit, Qodo, Ellipsis, and similar tools. 
But LLM review of LLM-generated code is still asking the same question. Can anyone predict production behaviour from a diff? **The answer is still no**. 

Shadow execution sidesteps the question entirely. Instead of asking "will this be correct?", you ask "is this already correct?" and let production traffic answer.
 
## Caveats
 
This pattern isn't universally applicable. Some constraints from the Scientist library's design still apply here.
 
* **Side effects are hard.** If the candidate has side effects (writing to a database, calling an external API, sending an email) then running it in shadow is dangerous. The experiment needs to be designed around a read-only or idempotent candidate, or the side effects need to be suppressed in the candidate path.
 
* **Result comparison isn't always obvious.** Comparing two floats, or two objects with timestamps, or two responses that are semantically equivalent but structurally different, requires custom comparison logic. The LLM would need to generate this too.
 
* **Latency overhead.** Running two code paths doubles the work per request, even if the candidate runs in a separate thread. For latency-sensitive paths this may be unacceptable for anything more than a short observation window.
 
* **The LLM needs system access.** For this to be fully LLM-orchestrated, the agent needs write access to the feature flag system, the ability to set timers or schedule future tasks, and a channel back to the human when the observation window closes. That's a meaningful expansion of the agent's footprint.
 
None of these are blockers. They're engineering problems worth surfacing so the pattern gets adopted where it fits, rather than everywhere by default.
 
## Conclusion
 
The PR review process has remained mostly unchanged even as code authorship has shifted to machines. The proposal here is to make the *review* commensurate with the *risk*, and to use the LLM not just as a code author but as the orchestrator of a structured, time-bounded production experiment that produces evidence rather than asking for faith.
 
The pieces all exist: GitHub Scientist and its descendants for the parallel run pattern, feature flag systems for controlling the rollout, automated cleanup agents for the teardown. What's missing is the workflow that connects them, with the LLM holding the thread from fix generation through to scaffolding removal.
 
That seems like the right shape for human-LLM collaboration on non-trivial production changes: the LLM does the work, sets up the evidence collection, and handles the cleanup; the human provides judgment at the moments that matter.
