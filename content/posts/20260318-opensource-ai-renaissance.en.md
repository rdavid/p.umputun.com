---
title: "The open source renaissance in the age of AI"
slug: opensource-ai-renaissance
date: 2026-03-18T15:17:55-05:00
draft: false
tags: ["geek stuff"]
---

![](/images/posts/opensource-ai-renaissance.png#floatright)

My point of view, which runs directly counter to what many popular thought leaders are saying, is that 2026 has become the year of open source renaissance. And it happened because the quality of work from contributors — the people who bring changes, suggestions, and even questions to projects — has noticeably improved with the mass adoption of agentic programming.

<!--more-->

## how things were before

I've been doing open source for over a decade, and my experience with external contributors is very much first-hand. I never really complained about people bringing wrong PRs or writing wrong tickets, though it slipped out occasionally. By and large, I didn't have major problems with participants in my projects even before 2026. Annoying people have always existed everywhere, and open source is no different from the rest of the world. I'd even say that my GitHub projects attracted noticeably fewer of them than I encounter in real life.

However, the quality of contributions was never consistent. It depended on the project, but the overall picture looked roughly like this: about 10% of proposals were the kind I'd eagerly grab and try to merge as quickly as possible. About 30% were complete nonsense. The remaining 60% required thoughtful work, significant revisions, and lengthy back-and-forth discussions.

## the shift

What happened over the last six months, and especially since the beginning of this year, is a fundamental shift. Across all my projects without exception, old and new, the number of contributors using AI to prepare pull requests, tickets, and problem descriptions has reached probably 90%, and in some projects a full 100%. It's a mass phenomenon and doesn't depend on the project's domain. You might think that AI-related projects naturally attract AI-savvy people, but the same thing applies to all the rest that were written long before LLMs became a household term.

This isn't a case of "the process seems to be starting" or "the process is underway." No, this train has already arrived, it's here, and we're probably seeing its final form. Outside of GitHub, I fully acknowledge there are huge numbers of programmers and companies that haven't made this transition yet, but open source contributors are clearly leading the charge and embracing AI as a real optimization of their effort and mental energy.

## how this helps the maintainer

I organize my own work with as much automation as possible too. When a PR or ticket comes in, everything starts automatically: a skill in Claude Code checks the proposal's adequacy not only from a technical standpoint but also from a logical one and from the perspective of the project's domain. Do we even need this, what problem does it solve, are there simpler approaches, is this even our problem to solve? All the questions I used to ask manually in tickets.

It's not a fully automatic process. After the analysis comes the hybrid part: I, as a Human with a capital H, look at what the AI wrote, ask clarifying questions, conduct additional research — all from Claude Code. I rarely need to dig into the code myself anymore. Then the AI and I draft a response, since it's rare for a PR to be immediately acceptable. Usually there are minor shortcomings, and AI is tremendously helpful in sorting them out, including stylistic inconsistencies. If my project has a certain coding style that I may have forgotten about but 90% of the code follows, it's nice when contributed code matches it too. And even those subtle style issues are something AI helps identify.

The agent's help in researching the history of an issue and finding similar past requests is also hard to overstate. Somewhere in the back of my mind I remember that something similar came up before, and I'd probably find it on my own, but why bother when it's part of the same skill? It automatically checks the history, looks at how I previously interacted with a given contributor, how much I harassed them with my demands and requirements, and all of this brings stability to the process on one hand and a tangible increase in my response speed on the other.

## response time

Those who actively contribute to my projects have probably noticed that my response time has improved dramatically. Previously, there was nothing unusual about a ticket sitting for a week, two, three, or sometimes months. I never treated this as work — it's a hobby. I'd come to tickets and PRs when I was in the mood and in the right mental state, and for some tickets that state didn't arrive for months, so they just waited for their time.

With the agentic approach, things that used to seem tedious and required deep immersion and switching mental context to an entirely different domain are now trivial. It's no longer a question of laziness and conserving effort but a matter of a few clicks and reading a couple of paragraphs of analysis that reliably trigger the right memories and load the necessary context into my head.

## contribution quality

As for those who bring me all this, the quality of their work has clearly improved, and most importantly it has become more consistent. The 10-30-60 split I mentioned earlier has now turned into roughly 50% of proposals that can be accepted right away, 40-45% that can be taken after minor refinements and clarifications, and about 5% that are complete nonsense and messages from a parallel universe.

That 5% is the most entertaining part. I'm not sure how people manage to produce such things — none of my own agents would write anything like it. I suspect it's because more and more people, armed with agents, feel like almost-real programmers and drive their AI without much understanding of what they're actually asking. Perhaps their prompts suffer. Perhaps they don't even bother cloning the project, and the AI hallucinates on a blank canvas based on the user's own hallucinations. The result is gems that seem to contain some kind of meaning, but any attempt to understand it is a sure path to madness: all the words make sense, the sentences make sense, but the overall meaning completely eludes you.

## contribution volume

Beyond quality, the sheer volume has grown noticeably too. In my recent projects, the number of contributors who bring genuinely useful ideas and solid PRs has increased dramatically. I can't give an exact figure, but it feels like at least two, maybe three to five times more than before.

The nature of contributions has changed as well. Previously, I'd often get contributors with general ideas, without any obvious practical need behind them. Maybe they had one, maybe not, but it looked more like an abstract "wouldn't it be nice if..." suggestion. Now it's different. Now the vast majority of changes are about specific problems that users actually experience. I think it's because now, with such a powerful tool as an AI agent at their disposal, people no longer settle for workarounds — they go ahead and propose actual fixes, improvements, and changes. And in most cases their proposals genuinely make sense.

## conclusions

To sum up, I'm glad about the improvement that agents have brought to open source. They've simplified and accelerated my work as a maintainer, and the overall quality of contributions has noticeably risen. If someone tells you it's the opposite, either don't believe them or their experience is very different from mine, though I have a sneaking suspicion the problem might be on their end. Perhaps it's a matter of scale: if a project has not thousands but tens of thousands of stars and not hundreds but thousands of contributors, sheer volume starts to become its own kind of problem, and those 5% of entertaining messages start to get annoying. For me they don't yet, and I believe that the overall improvement in contribution quality is well worth those few percent that occasionally irritate and drive you a little crazy.

_This post was translated from the [original](/2026/03/18/opensource-ai-renaissance/) with AI assistance and reviewed by a human._
