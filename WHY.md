# Why Auto-Context exists

## The AI programming agent experience today

If anyone has ever started a greenfield project from scratch using an AI agent, I'm sure we are all aware of this pattern: at the beginning the agent churns out code like there's no tomorrow. The speed with which it cranks out high-quality code starting from zero is groundbreaking and it's probably the best tool to learn a new framework by working with it.

However, as projects become more complex, the current AI agents start to lose the plot. They introduce bugs by assuming that old rules for the project are no longer needed, and they struggle with understanding what is important and what is to be changed in a task. Even when using the new standard of making another agent review the code before considering it complete, agents want to produce a quick win so much that they often introduce new bugs or mess up the old code in subtle ways.

## Think of a senior software developer

Think of a senior software developer who has worked on a project for 5 years. In that 5 years they wrote (and re-wrote) a significant part of the project, and generally learned a lot about where everything is, what parts of the system are problematic and how bugs occur. They largely know immediately how an architecture change would affect the project — which parts would be affected and what are the possible pain points a project can suffer with.

I understand that reading the code also explains the code, but the main idea here is that these people _know_ these things without having to read a single line of code, just because they learned these things over the course of years.

AI agents do not have these capabilities out of the box, as normally they are required to read code to understand code and have no knowledge of the project beforehand.

## CLAUDE.md

A lot of these previous knowledges are solved by a dedicated CLAUDE.md file: a file like that would contain best practices for the repo, but it's important to understand how agents use this information. The agent is able to read and process code in the codebase fast, even utilizing git history to understand how different parts of the system evolved over time.

What agents are unable to understand is anything external that is happening outside of the codebase. For example: coding styles that are chosen because the underlying hardware demands it. Decisions that were made to make sure integrations with other systems do not break. Even a good CLAUDE.md file is not expected to answer these things, but in the long-term these whys are as important as all other details in the codebase.

Code comments are great, but often those are not the right place to store information like this. It's nice to keep these why-s close to the code affected, but finding the proper whys and understanding the whole picture is hard if all the functions' comments need to be read and understood.

Unit tests are the best way of handling these issues but it's also worth mentioning how verbose those can get. What we want is something that is easy to look up and understand, lives in the agent's context whenever the thing is needed and gives ideas about what is important in the project.

## A rolling document, kept in sync like a linter

The easiest way of solving this is to have a rolling document explaining the whys of the project and pointing to the right set of files to read for an agent. It should help the agents find the proper files, understand the proper patterns and critical paths the program needs to have in order to work well continuously.

As with any software documentation, the challenge is to keep it up-to-date always. Before agents it was harder to see the value of updated docs, but with the advent of agents the need to keep these updated is clear. Treat it like a linter — another step to run before committing to keep the project upkeep costs the minimum.

That's what Auto-Context does. It runs on every PR and push, analyzes what changed, and updates the context files so your agents never drift out of sync with the codebase.
