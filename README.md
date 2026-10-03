# Awesome AZ-104

A short, read-only study guide for **Exam AZ-104: Microsoft Azure Administrator**.

No labs. No deployments. No cost. Just what you need to understand and remember to pass.

> Based on the Microsoft AZ-104 skills measured **as of April 17, 2026**.
> Source of truth: [Microsoft study guide](https://learn.microsoft.com/en-us/credentials/certifications/resources/study-guides/az-104).

## Who is this for

Engineers who already know some Azure and want a fast, exam-focused pass over everything AZ-104 tests, including the traps.

## What every topic answers

1. **What is it?**
2. **When do I use it?**
3. **What could the exam trick me about?**

## Layout

| Domain | Weight (official) | Folder |
|---|---|---|
| Manage Azure identities and governance | 20–25% | [01-identities-governance](01-identities-governance/README.md) |
| Implement and manage storage | 15–20% | [02-storage](02-storage/README.md) |
| Deploy and manage Azure compute resources | 20–25% | [03-compute](03-compute/README.md) |
| Implement and manage virtual networking | 15–20% | [04-networking](04-networking/README.md) |
| Monitor and maintain Azure resources | 10–15% | [05-monitoring-maintenance](05-monitoring-maintenance/README.md) |

Also:

- [EXAM-GUIDE.md](EXAM-GUIDE.md): verified exam facts and how to prepare
- [QUICK-REVISION.md](QUICK-REVISION.md): one-pass review for the last days
- [EXAM-DAY-CHECKLIST.md](EXAM-DAY-CHECKLIST.md): how to read questions, last-hour list
- [references/microsoft-resources.md](references/microsoft-resources.md): official links and the objective-to-section coverage map

## Recommended study order

1. [EXAM-GUIDE.md](EXAM-GUIDE.md) (10 minutes)
2. Identities and governance, then **Networking** (everything else depends on it), then Storage, Compute, Monitoring
3. Re-do the **Quick Check** questions in each domain
4. Take Microsoft's free [practice assessment](https://learn.microsoft.com/en-us/credentials/certifications/exams/az-104/practice/assessment?assessment-type=practice&assessmentId=21)
5. Read [QUICK-REVISION.md](QUICK-REVISION.md) twice in the final week
6. Read [EXAM-DAY-CHECKLIST.md](EXAM-DAY-CHECKLIST.md) the day before

## How to read a domain file

Each topic has the same blocks:

- **What is it?** plain explanation and a mental model
- **Know for AZ-104**: only what the exam needs
- **Exam trap**: marked with `[!WARNING]`
- **Remember**: one-line takeaways

Each domain ends with **Know This** (memorize), **Understand This** (reason about it), comparison tables, scenarios and quick checks.

## Scope

This is a conceptual guide. Hands-on practice helps, and Microsoft recommends it, but it is deliberately outside this repo. Commands appear only where they clarify a concept.

Quick Check questions are original. They are **not** real exam questions.

## Found an error?

Azure changes often. Open an issue and link the Microsoft Learn page that shows the correct behavior.
