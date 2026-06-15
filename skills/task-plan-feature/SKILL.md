---
name: task-plan-feature
description: This skill conducts an interview with the user about what they want to build.
---

Interview me relentlessly about a plan, feature, or design until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

If a question can be answered by exploring the codebase, explore the codebase instead.

Always use the interview tool for your questions.

## Context

In order to understand intent, try the following in order (only use the first valid option):

1. `TASK.md` in the project root (if it exists)
2. Current conversation history (if you have provided at least one response)
3. Ask directly what you're interviewing me about.

## During the interview session

### Challenge against the glossary

When I use a term that conflicts with the existing language or terminology currently in us, call it out immediately e.g. "Your project defines 'cancellation' as X, but you seem to mean Y — which is it?"

### Sharpen fuzzy language

When I use vague or overloaded terms, propose a precise canonical term e.g. "You're saying 'account' — do you mean the Customer or the User? Those are different things."

### Discuss concrete scenarios

When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about the boundaries between concepts.

### Cross-reference with code

When I state how something works, check whether the code agrees. If you find a contradiction, surface it e.g. "Your code cancels entire Orders, but you just said partial cancellation is possible — which is right?"

## Closing tasks

Once you have completed the interview, write a comprehensive, todo-driven plan in `SPEC.md` at the project root that can then be used to build what is required.

The `SPEC.md` will not be reviewed by humans, so it should be constructed in a way that best serves the AI model implementing it.

Do not attempt to actually implement `SPEC.md`. Your job is simply to write it.
