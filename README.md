# churn-risk-scan

Kortix is the open-source CompanyOS: your agents, their skills, your company memory and every connector in one git repo you own, with the agents working on real cloud computers.

This repository is a working churn-risk scanner for Kortix. It holds the three files that make the scan run — an agent, the scoring skill and the daily trigger — and nothing else. Clone it, point it at your own product database, support tool and billing system, and it posts a ranked at-risk list to Slack every morning.

It is the code behind the pattern in [How we flag at-risk accounts](https://kortix.com/use-cases/churn-risk). Read that page if you want the walkthrough with screenshots; read this repository if you want the files.

## What it does

Once a day, a cron trigger starts a session. The session reads four signals per account, scores them as a combination rather than one at a time, and posts one ranked list to a Slack channel:

```
Daily churn-risk scan · 06:00

Flagged 8 accounts. 2 are highest risk: usage down 34% in 30 days, an
escalated support thread, and a renewal in the next 30 days. For both,
the suggested next step is a renewal conversation this week.
```

That output is the example run printed on the Kortix use-case page. It is one team's run, not a benchmark and not a customer result.

The four signals:

| Signal | Source | What counts |
| --- | --- | --- |
| Usage | Your product database (Postgres) | Core activity down 20% or more against the prior 30 days; dormant after 14 days with almost no activity |
| Support friction | Plain | Thread volume above the account's own history, or threads that stay unresolved; cancellation language counts as escalated |
| Billing | Stripe | A missed or failed payment |
| Renewal | Billing / Stripe | A renewal inside the next 30 days |

## The scoring rule that makes it useful

No single signal is high risk on its own. An account whose usage dipped might be a seasonal customer; an account whose usage dipped, whose support threads got angrier and who renews next month is a different conversation. The skill ranks accounts with all of: declining usage, rising support friction, and an imminent renewal or a failed payment.

The scoring lives in `skills/churn-signals/SKILL.md`, as text a person can read and a reviewer can diff. The thresholds are not hidden in a model.

## Read-only, on purpose

The agent reads Postgres, Plain and Stripe. Its only write is the Slack post. It does not send email, reply to a ticket, change a subscription or contact a customer. Every listed account carries a reason and one suggested next step, and a person on customer success decides what to do with it.

On Kortix the connector credentials are brokered server-side and never enter the agent's machine, and each run happens in its own isolated sandbox. Nothing about the customer data leaves the systems you already govern.

## Use it

1. Install the CLI and sign in:

   ```sh
   curl -fsSL https://kortix.com/install | bash
   kortix login
   ```

2. Create a project and drop these files in:

   ```sh
   kortix init churn-risk
   cd churn-risk
   cp -r /path/to/churn-risk-scan/{agents,kortix.yaml} .
   mkdir -p skills && cp -r /path/to/churn-risk-scan/skills/churn-signals skills/
   ```

   Or use the setup prompt on [the use-case page](https://kortix.com/use-cases/churn-risk) and let the agent write the same files and open a change request for you to review.

3. Connect the tools. Open **Connectors** in the project and connect your Postgres, Stripe and Slack accounts, then add your Plain API key as `PLAIN_API_KEY` under **Secrets**. Credentials are brokered server-side and never enter the agent's machine.

4. Grant the connectors to the `churn-risk` agent, run one scan by hand, and check the ranking against what your team already knows before the schedule takes over.

   ```sh
   kortix sessions new --agent churn-risk --prompt "Run one churn-risk scan. Read only."
   kortix ship
   ```

5. From then on the `daily-churn-scan` trigger in `kortix.yaml` starts the agent every day at 06:00 UTC. Change the cron line to move it, or edit it under **Triggers**.

## What is in here

```
kortix.yaml                    manifest: the agent, the skill, the daily trigger
agents/churn-risk.md           the agent's instructions: its job, its inputs, its one output
skills/churn-signals/SKILL.md  the four signal rules and the ranking rule
```

Every change to those files is a git commit. On Kortix an agent's change to the project lands as a change request a person merges, so the scoring rules cannot drift silently.

## Reading the other side of the scan

The files here are the operator's view: the rules, the trigger, the one output. The buyer's view of the same job is written up separately, with the numbers a team argues over. Start with [what churn prediction means for an account list](https://churnpredictionguide.com/what-is-churn-prediction) and the [signal list this skill implements](https://churnpredictionguide.com/churn-signals). [What a scoring run looks like](https://churnpredictionguide.com/churn-risk-scoring) and [what the agent version changes](https://churnpredictionguide.com/ai-churn-prediction-agent) cover the output and the automation. If your customer data cannot leave your own infrastructure, read [what self-hosting changes about the data](https://churnpredictionguide.com/self-hosted-churn-prediction) before you connect a production database.

## Licences and attribution

The files are yours to adapt. The pattern is Kortix's use case, documented at [kortix.com/use-cases/churn-risk](https://kortix.com/use-cases/churn-risk). Kortix itself is open source; the CLI, the manifest reference and the self-host path are at [kortix.com/docs](https://kortix.com/docs).
