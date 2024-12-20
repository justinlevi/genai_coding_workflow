# Coding & Development in the Era of GenAI

Coding is evolving, and it's becoming easier to turn ideas into reality. I think the future of coding will be about using workflows to get things done. It will be less about being an expert in a specific framework, library, or system. Knowing strategies and design patterns will matter more than learning a programming language. It will be essential to know what makes a system "good" and how to build a flexible architecture. You'll also need to distinguish when a solution limits your options versus when it helps. 

By now, you've likely seen an LLM create a working to-do list app or a playable Flappy Birds clone with a simple prompt. This is impressive and seems magical. As codebases grow, using an LLM to add features or fix issues can become challenging. A more methodical approach can be helpful. 

The following is very much a work in progress and a living document. Many of the ideas come from videos and posts from these accounts:

- Mckay Wrigley: https://x.com/mckaywrigley

- Rohan Paul: https://x.com/rohanpaul_ai

- Ray Fernando: https://x.com/RayFernando1337

- Echo.hive: https://x.com/hive_echo

---

We can boil the problem down to a few key issues:

1. Lack of a good plan

2. Scope of the task is too big.

3. No benchmarks or tests along the way to prove success.

4. No iterative process for leveraging learnings for future tasks.

## The System

Step 1: Planning or Scoping the problem:  Are you able to articulate what it is you’re trying to accomplish? What is the task? What are the goals?  Sending too much info for the LLM to sift through to figure out what you’re asking it to focus on or do is a sure fire way of having the LLM create something tangential to what it is you’re trying to accomplish. This leads to another whole mess of having to decide whether or not it’s worth trying to unwind the mess and get rid of the kruft that the LLM created, or scrap it all and do it yourself. 

The solution: Build a Project Requirement Document (PRD)  - use an LLM to help you build this. Iterate, this is your roadmap. 

What to include:

- Your directory structure

- The task description

- Break it down into the 5 why’s

- Break it down into micro steps

- Provide specific rules, libraries, code examples

PRD Creation (to be modified for your specific task/goal)

prompt:

```

# PRD Generation

Please create a comprehensive Product Requirements Document (PRD) for [TOPIC]. Structure the document following these guidelines:

## Required Sections

### 1. Overview

- Provide a high-level summary of the [TOPIC]

- Define the scope and objectives

- Identify key stakeholders

- State the business value and impact

### 2. Protocol Steps

For each major component or phase:

- Define clear steps and procedures

- Specify entry and exit criteria

- List dependencies and prerequisites

- Document required tools or resources

### 3. Testing & Validation

Detail the approach for:

- Test execution procedures

- Success criteria

- Error handling

- Documentation requirements

### 4. Documentation Requirements

For each identified issue or change:

- Problem statement and impact

- Technical analysis and root cause

- Proposed solutions with justification

- Prevention strategies and best practices

### 5. Implementation Guidelines

Include:

- The five why's for this step

- Step-by-step procedures

- Required validations

- Success criteria

- Rollback procedures

### 6. Response Format

For each iteration:

- Current status reporting format

- Test result documentation

- Update procedures

- Action plan structure

- Documentation update requirements

## Format Requirements

1. Use clear hierarchical structure with proper Markdown formatting

2. Include specific examples where applicable

3. Define measurable success criteria for each component

4. Provide troubleshooting guidelines

5. Include template sections for standard responses

## Additional Guidelines

- Use bullet points for lists longer than 3 items

- Include tables for complex comparisons

- Add code blocks for command examples

- Define all technical terms

- Cross-reference related sections

- Include version control procedures

## Important Notes

- Maintain comprehensive documentation

- Focus on clarity and completeness

- Ensure traceability

- Keep format consistent

- Include all necessary metadata

Please ensure all sections are thorough while maintaining clarity and usability. The PRD should serve as both a planning document and a reference guide for implementation.

```

PRD Review 

prompt: 

```

You are an experienced technical lead evaluating a Product Requirements Document (PRD). Your goal is to ensure the document is complete and implementable by a junior developer. Analyze the PRD using these criteria:

Technical Clarity:

1. Are all technical terms clearly defined?

2. Is each feature described with specific, measurable requirements?

3. Are all system interactions and dependencies explicitly mapped?

4. Are performance requirements and constraints quantified?

Implementation Readiness:

1. Are acceptance criteria provided for each feature?

2. Are edge cases and error scenarios documented?

3. Is the required tech stack and architecture specified?

4. Are there clear API specifications where needed?

Developer Guidance:

1. Are implementation prerequisites listed?

2. Is there sufficient context about existing systems?

3. Are there examples or references for complex features?

4. Is the scope clearly bounded with what's out of scope?

Data Requirements:

1. Are all data models and schemas defined?

2. Is data flow between components documented?

3. Are security and privacy requirements specified?

4. Is data validation criteria provided?

For each section, provide:

- A completeness score (1-5)

- List of missing critical information

- Specific questions that need clarification

- Recommendations for improvement

Your evaluation should highlight both strengths and gaps, with particular focus on areas that could block or confuse junior developers.

```

Iterate on the PRD using LLMs like 01 or an LLM that is very good at reasoning

prompt:

```

I have the following PRD

[ COPY/PASTE PRD HERE ]

==========

Please break down each implementation phase into much smaller more granular and detailed steps so that we could hand of this work to a junior programmer. focus on best practices and try to anticipate potential "gotchas" in the implementation and testing. We are trying to use modern approaches. Do not change the format or lose any information/details, just add to the existing document or modify is something is not correct. 

```

[EXAMPLE of a "final" PRD Prompt](./PRD.md)

Implement a TDD(ish) approach first - Create a Testing markdown file that describes the style and guidance around testing you’re interested in. Adding things like  “testing behavior, not implementation details” in software development. This approach emphasizes writing tests that validate the observable behavior of a system rather than its internal workings. By focusing on what the system does, rather than how it does it, tests become more robust and less susceptible to breaking due to internal code changes that don't affect functionality.

For instance, instead of asserting that a specific method is called (an implementation detail), you would assert the expected outcome of that method's execution. This ensures that tests remain valid even if the internal implementation changes, as long as the external behavior stays consistent. Also build a development guide document or .cursor rules doc

The solution: Create a Testing document to be used for a specific task/phase implementation later on. 

[Testing Document Example](./testing.md)

Narrowing your task or goal - Is there a very narrow and clear task or plan to solve - Just like a team without a project manager, product owner, a developer without a clear plan or set of steps needed to achieve a goal, an AI will struggle to get the job done. (Side note: may be useful here thinking of an LLM as just another colleague or employee that you need to provide really specific instructions to get the job done) There is also the problem of the LLM not being able to adequately keep track of the entire application code base, files, classes, etc that all are players in the final desired result (side note: this is an argument against over engineering into too many abstract classes, files too soon.)

(Relevant thread on Reddit thread on torturing your models with context spam: https://www.reddit.com/r/LocalLLaMA/comments/1hh2lfc/please_stop_torturing_your_model_a_case_against/) 

The solution: reference your PRD and have the LLM focus on a specific phase and step

prompt:

```

@prd.md 

I would like you to follow the  [TITLE OF PRD HERE] PRD. Please first review and then start working through each phase incrementally making sure all tests pass before and after each phase. 

Before adding any new file or functionality, ensure the functionality does not already exist in the application. Follow the structure of the application exactly or ask questions if you're not 100% sure. Ensure all tests follow the guidelines and best practices as defined in @tests.md 

Please start on the tests and implementation for phase [X]

```

Post Mortem/Review - 	after a task or phase completion, analyze and compare planned versus actual implementation, identify what wasn’t completely clear in the plan, what could have been improved, and documenting lessons learned. Then applying these learnings back to the original plan both on the phase/step that was completed and the next phases/steps to come. 

```

Review the provided implementation of [PRD Name] Phase [X], analyzing the git diff and existing codebase. Evaluate against these criteria:

Adherence to PRD:

1. Does implementation fulfill all Phase [X] requirements?

2. Are there deviations from PRD specifications? If so, are they justified?

3. Is implementation properly scoped to current phase?

Code Quality:

1. Are best practices followed for chosen tech stack?

2. Is code maintainable and well-documented?

3. Are there potential performance or scalability issues?

4. Is error handling comprehensive?

Integration & Dependencies:

1. Does implementation properly integrate with existing systems?

2. Are dependencies appropriate and minimized?

3. Could implementation choices impact future phases?

Provide:

1. Implementation Strengths: Key decisions and approaches done well

2. Areas for Improvement: Specific issues or concerns, ordered by priority

3. Forward-Looking Considerations: Potential impacts on future phases

4. Specific Recommendations: Actionable improvements with examples

Focus analysis on correctness, maintainability, and scalability rather than style preferences.

```

Apply Learnings

prompt:

```

Given what we learned in Phase X and Phase X, is there anything we should review and add for Phase X that would dramatically improve the likelihood of success for the next phase? 

```

Continuation

prompt:

```

@prd.md @testing.md

Please first review the PRD and then continue working through each phase incrementally making sure all tests pass before and after each phase. 

Before adding any new file or functionality, ensure the functionality does not already exist in the application. Follow the structure of the application exactly or ask questions if you're not 100% sure where something should live. Ensure all tests follow the guidelines and best practices as defined in @tests.md 

Please start on the tests and implementation for: [PHASE X]

Run all tests before and after the implementation

```

## Summary

In review:

1. Planning/Scoping the problem - create a detailed PRD and testing document

2. Implementation: - TDD First  micro steps

3. Post Mortem: - Have the llm leverage your learnings and apply back to the PRD.

4. Continue: - Move on to the next phase/step

Tools

* Cursor

    * Claude Sonnet - primary code gen

    * 01 - reasoning through sticky issues and entire code bases

* Cursor extension

    * Spec Story - https://marketplace.visualstudio.com/items?itemName=SpecStory.specstory-vscode

I've also been using this tool quite a bit lately for concat'ing entire repos to use for LLM coding tasks. My fork has a CLI tool versus using the web app

https://github.com/justinlevi/gitingest
