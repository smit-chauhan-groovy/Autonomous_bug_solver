# Hackathon Pitch Script (3 Minutes)

> **Goal:** Strictly 3 minutes. Focus on Client Problem, Give a Live Demo, Explain KRAs, and End with Impact. Do NOT reveal actual client names.

---

## 1. Introduction & Client Problem (45 seconds)
"Hi judges. On our recent client project—a large-scale SaaS platform—we noticed a massive bottleneck. Our team was spending over 15 hours a week just doing routine bug fixes."

"When a bug comes in, a developer has to stop feature work, read the ticket, hunt down the context across a massive codebase, write the fix, test it, and write a PR description. It takes about 2 to 3 hours just to fix a single minor issue."

"So, we built the **Autonomous Bug Solver**. Give it a ticket ID, and a multi-agent orchestrated system automatically fetches the context, writes the fix, tests it, and opens a Pull Request. Completely autonomously."

## 2. Show, Don't Tell (Live Demo) (1 minute 30 seconds)
"Let’s look at it live. Here is an active bug ticket in our system. I am going to trigger the agent."

*(Start the Orchestrator script)*

"While it runs, here is what’s happening behind the scenes:
1. The **Linear Fetch Agent** is reading the bug description.
2. The **Context Agent** and **File Locator Agent** are scanning the codebase to find the exact files that are broken.
3. The **Reasoning & Fix Agents** are generating the code solution.
4. And finally, the **PR Agent** is pushing the branch and generating a Pull Request."

*(Show the terminal output running autonomously. When the PR link pops up, click it and show the judges the generated code and PR description.)*

"As you can see, the code is fixed, and the PR is ready for human review."

## 3. Explain the KRAs (30 seconds)
"Our success here is measured across a few Key Result Areas (KRAs):
1. **Speed:** We reduced Mean Time to Resolution for minor bugs from 3 hours to under 5 minutes.
2. **Quality:** We measure the CI/CD pipeline pass rate of the agent's code, aiming for an 85% first-try pass rate.
3. **Autonomy:** We track how many 'Zero-Touch PRs' the agent creates, where a human only has to click 'Approve'."

## 4. End with Impact (15 seconds)
"By automating the grunt work of context-gathering and minor bug fixing, this agent **saves our engineering team over 60 hours a month**. That is 60 hours returned to building actual features instead of chasing console errors."

"Thank you. We are ready for your questions."
