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

---

### 1. Planning and Scoping

LLMs will gladly create text for you, the goal is to get the LLM to create meaningful text scoped to the goal. When trying to have an LLM write code for you, sending too much context to sift through is a good way to have the LLM create something close to what you're looking for, but just a bit off. This leads to another mess of needing to decide whether or not it’s worth unwinding the mess and getting rid of the kruft that the LLM created, or scrap it all and do it yourself. 

The main questions we need to address.

* what it is you’re trying to accomplish? 
* What are the steps required to accomplish this? 
* What is the success criteria, what can we measure?  

**The solution:** Build a Project Requirement Document (PRD) prompt - use an LLM to help you build this. Iterate and improve this document as much as you can because this will be your roadmap. 

Here's a few things you will want to include in your final PRD:
1. Your directory structure
2. A detail task description
3. Each task should be broken down into the 5 why’s
4. Each phase/task should be broken down into testable micro steps
5. Provide specific rules, libraries, code examples


#### PRD Creation

Here are the series of prompts I will use to generate and iterate on my PRD:
<details>
<summary>1. PRD Creation Prompt: (Click to expand/collapse)</summary>

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
- The five why's for each step
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
</details>

---

Next I use an LLM to review the PRD and provide feedback on what is missing or unclear.



<details>
<summary>2. PRD Review Prompt: (Click to expand/collapse)</summary>

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
</details>

---

Use a good reasoning LLM like 01 to add additional details to the PRD.

<details>
<summary>3. PRD Iteration Prompt: (Click to expand/collapse)</summary>

```

I have the following PRD

[ COPY/PASTE PRD HERE ]


==========

Please break down each implementation phase into much smaller more granular and detailed steps so that we could hand of this work to a junior programmer. focus on best practices and try to anticipate potential "gotchas" in the implementation and testing. We are trying to use modern approaches. Do not change the format or lose any information/details, just add to the existing document or modify is something is not correct. 
```

</details>

---


["Final" PRD Example](./PRD.md)
This was created for generating a database integration


### 2. Testing Guidelines

I have found that one effective way to get the LLM to implement a feature effectively is to leverage the best practices found in Test Driven Development (TDD). In order to do this, we first need to create a set of testing guidelines.

**Testing Guidelines** - Using a similar process from above, you can then create a testing markdown file that describes the style and guidance around testing you’re interested in. Adding things like  “test behavior, not implementation details” can be helpful to steer the LLM in the right direction. This approach emphasizes writing tests that validate the observable behavior of a system rather than its internal workings. By focusing on what the system does, rather than how it does it, tests become more robust and less susceptible to breaking due to internal code changes that don't affect functionality.

For instance, instead of asserting that a specific method is called (an implementation detail), you would assert the expected outcome of that method's execution. This ensures that tests remain valid even if the internal implementation changes, as long as the external behavior stays consistent. 


[Testing Document Example](./testing.md)
Feel free to use this as a starting point and modify it to fit your needs.


### 3. (Optional) .cursorrules file
My preferred way ide is cursor and using a .cursorrules file. This is a great way to set up your cursor environment and provide a consistent way to interact with the LLM. You can think of this file as a system prompt for your project. Cursor will use this file to guide the LLM.


<details>
<summary>.cursorrules file: (Click to expand/collapse)</summary>

```

  You are an expert in [X], [Y], and [Z].
  
  Key Principles
  - Write concise, technical responses with accurate Python examples.
  - Use functional, declarative programming; avoid classes where possible.
  - Prefer iteration and modularization over code duplication.
  - Use descriptive variable names with auxiliary verbs (e.g., is_active, has_permission).
  - Use lowercase with underscores for directories and files (e.g., routers/user_routes.py).
  - Favor named exports for routes and utility functions.
  - Use the Receive an Object, Return an Object (RORO) pattern.
  - Favor composability over inheritance.

  
  Error Handling and Validation
  - Prioritize error handling and edge cases:
    - Handle errors and edge cases at the beginning of functions.
    - Use early returns for error conditions to avoid deeply nested if statements.
    - Place the happy path last in the function for improved readability.
    - Avoid unnecessary else statements; use the if-return pattern instead.
    - Use guard clauses to handle preconditions and invalid states early.
    - Implement proper error logging and user-friendly error messages.
    - Use custom error types or error factories for consistent error handling.
  
  Dependencies
  - [ PROJECT SPECIFIC DEPENDENCIES ] 
  
  [ FRAMEWORK SPECIFIC GUIDELINES ]
  
  Performance Optimization
  - Minimize blocking I/O operations; use asynchronous operations for all database calls and external API requests.
  - Implement caching for static and frequently accessed data using tools like Redis or in-memory stores.
  - Optimize data serialization and deserialization with Pydantic.
  - Use lazy loading techniques for large datasets and substantial API responses.
  

## Test-Driven Development Approach
Each phase must have passing tests before proceeding to the next phase. The testing strategy follows:

1. Write tests first
2. Implement minimum code to pass tests
3. Refactor while maintaining test coverage
4. Document test cases and coverage


## Guidelines and Best Practices

- Always create modular and reusable code
- Always favor simplicity over complexity
- Composability over inheritance
- Document code to make it easier to understand
- Use meaningful variable and function names
- Use design patterns to make the code more readable and maintainable
- Use dependency injection to make the code more testable and maintainable
- Other design patterns which we should focus on are:
    - Dependency Injection
    - Inversion of Control
    - SOLID principles
    - KISS principle
    - YAGNI principle
    - DRY principle
    - Single Responsibility Principle
    - Open/Closed Principle
    - Liskov Substitution Principle
    - Interface Segregation Principle


## Core Testing Principles
Testing FastAPI applications requires a systematic approach focused on ensuring reliability and maintainability. Here are the fundamental principles for writing effective tests:

### Test Structure

#### Test Isolation
- Each test should be completely independent and not rely on the state of other tests
- This isolation makes tests easier to debug and maintain
- Use fixtures and setup/teardown to ensure clean test environments

#### Test Organization 
- Tests should be organized in separate files from the main application code
- Follow naming convention `test_*.py` for test files
- Group related tests into test classes or modules
- Use descriptive test file names that indicate what is being tested

### Essential Test Types

#### Unit Tests
Write focused tests for individual components and functions that:
- Verify specific functionality in isolation
- Test one concept per test function
- Use clear, descriptive test names
- Mock external dependencies
- Focus on testing business logic

#### Integration Tests
Create tests that validate:
- Interactions between components
- External service integrations 
- Complete request-response cycles
- Database operations
- API endpoint behaviors

The universally accepted structure for tests follows the "Arrange-Act-Assert" (AAA) pattern, which creates clear and maintainable test cases.

ALWAYS - make the minimum changes possible when refactoring, then re-test.


The PRD files you should be aware of are here:
[ABSOLUTE PRD DIRECTORY]

You should be working within this directory:
[ABSOLUTE PROJECT DIRECTORY]


You will use the following command to run the tests:
[TEST COMMAND]


```
</details>


### 3. Implementation

**Focus on a specific phase or task** - Next I will use the PRD we created earlier, along with the testing file, and start working on the specific phase or task. 

I currently use cursor composer in agent mode as my preferred way to implement the code. 

task prompt:
```
[INCLUDE THE PRD FILE HERE] @PRD.md 

[INCLUDE THE TESTING FILE HERE] @testing.md

I would like you to follow the  [TITLE OF PRD HERE] PRD. 
Please first review and then start working through each 
phase incrementally making sure all tests pass before and 
after each phase. 

Before adding any new file or functionality, ensure the 
functionality does not already exist in the application. 
Follow the structure of the application exactly or ask questions 
if you're not 100% sure. Ensure all tests follow the guidelines 
and best practices as defined in @tests.md 

Please start on the tests and implementation for phase [X]
```

After executing the above prompt, cursor will create or update the files necessary to complete the phase/step. 


**Post Mortem/Review** - Next, I will use an LLM to review the code that was created. We want to analyze and compare what was planned versus the actual implementation, identify what wasn’t completely clear in the plan, what could have been improved, and documenting lessons learned. Then applying these learnings back to the original plan both on the phase/step that was completed and the next phases/steps to come. 


Next I will execute the following prompt to review the code:
```
Review the provided implementation of [PRD Name] Phase [X], 
analyzing the git diff and existing codebase. Evaluate against 
these criteria:

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

**Apply Learnings**

Here is the prompt I use to apply learnings back to the PRD.

```
Given what we learned in Phase X and Phase X, is there anything 
we should review and add for Phase X that would dramatically 
improve the likelihood of success for the next phase? 
```

**Continuation on the next phase/step**

When I'm ready to continue on the next phase/step, I will execute the following prompt:
```
@prd.md @testing.md

Please first review the PRD and then continue working through 
each phase incrementally making sure all tests pass before and 
after each phase. 

Before adding any new file or functionality, ensure the
functionality does not already exist in the application. 
Follow the structure of the application exactly or ask questions
if you're not 100% sure where something should live. Ensure all 
tests follow the guidelines and best practices as defined in 
@tests.md 

Please start on the tests and implementation for: [PHASE X]

Run all tests before and after the implementation

```

## Summary


In review, below is the outline of the process I use to implement a feature with the assistance of an LLM.

1. Planning/Scoping the problem - create a detailed PRD and testing document
2. Implementation: - TDD First  micro steps
3. Post Mortem: - Have the llm leverage your learnings and apply back to the PRD.
4. Continue: - Move on to the next phase/step


### AdditionalTools

* Cursor (https://www.cursor.com/)
    * Claude Sonnet - primary code generation
    * GPT 01 - Reasoning through sticky issues and entire code bases
* Cursor extension - useful for saving and loading cursor sessions
    * Spec Story - https://marketplace.visualstudio.com/items?itemName=SpecStory.specstory-vscode



The following is a useful tool for concatenating entire repos to use for LLM coding tasks.
https://github.com/justinlevi/gitingest
