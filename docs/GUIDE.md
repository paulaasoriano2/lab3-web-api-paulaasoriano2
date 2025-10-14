---
title: "Web Engineering 2025-2026"
subtitle: "Lab 3: Complete a Web API"
format:
  html:
    toc: true
    toc-depth: 3
    number-sections: true
    code-fold: true
    code-tools: true
    code-overflow: wrap
    theme: cosmo
    css: |
      body { font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
      .quarto-title-block { border-bottom: 2px solid #2c3e50; padding-bottom: 1rem; }
      h1, h2, h3 { color: #2c3e50; }
      pre { background-color: #f8f9fa; border-left: 4px solid #007acc; }
      .callout { border-left: 4px solid #28a745; }
      /* Wrap long lines in code blocks */
      pre, code { white-space: pre-wrap; word-break: break-word; overflow-wrap: anywhere; }
  pdf:
    documentclass: article
    classoption: [11pt, a4paper]
    toc: true
    toc-depth: 3
    number-sections: true
    geometry: [margin=2.5cm, headheight=15pt]
    fontsize: 11pt
    linestretch: 1.15
    colorlinks: true
    breakurl: true
    urlcolor: blue
    linkcolor: blue
    citecolor: blue
    hyperrefoptions:
      - linktoc=all
      - bookmarksnumbered=true
      - bookmarksopen=true
    header-includes:
      - |
        \usepackage{helvet}
        \renewcommand{\familydefault}{\sfdefault}
        \usepackage{hyperref}
        \usepackage{fancyhdr}
        \pagestyle{fancy}
        \fancyhf{}
        \fancyhead[L]{Web Engineering 2025-2026}
        \fancyhead[R]{Lab 3: Complete a Web API}
        \fancyfoot[C]{\thepage}
        \renewcommand{\headrulewidth}{0.4pt}
        \usepackage{microtype}
        \usepackage{booktabs}
        \usepackage{array}
        \usepackage{longtable}
        \usepackage{xcolor}
        \definecolor{sectioncolor}{RGB}{44,62,80}
        \usepackage{sectsty}
        \allsectionsfont{\color{sectioncolor}}
        % Wrap long lines in code blocks for PDF output
        \usepackage{fvextra}
        \fvset{breaklines=true, breakanywhere=true}
        \DefineVerbatimEnvironment{Highlighting}{Verbatim}{breaklines,breakanywhere,commandchars=\\{\}}
lang: en
---

Welcome to the third lab assignment of the 2025--2026 course! This guide will help you complete the assignment efficiently. Although this guide is command-line oriented, you are welcome to use IDEs like **VS Code**, **IntelliJ IDEA**, or **Eclipse**—all of which fully support the tools we'll be using. Ensure you have at least **Java 21** installed on your system before getting started.

**Estimated time**: 2 hours.

## System requirements

For this assignment, we'll be using the following technologies:

1. **Java Version**: The project targets **Java 21** (toolchain).
2. **Programming Language**: [Kotlin 2.2.10](https://kotlinlang.org/).
3. **Framework**: [Spring Boot 3.5.3](https://docs.spring.io/spring-boot/).
4. **Build System**: [Gradle 9.0.0](https://gradle.org/).
5. **Code Quality**: **Ktlint** Gradle plugin for code formatting.

## Getting started

### Clone the repository

1. Clone your Lab 3 repository and change into the directory:

```bash
git clone https://github.com/UNIZAR-30246-WebEngineering/lab-3-restful-ws-<your-github-user>.git
cd lab-3-restful-ws-<your-github-user>
```

2. Run tests:

```bash
./gradlew test
```

## Primary Task

Complete and verify the tests in `ControllerTests` that exercise the `EmployeeController` REST API. The controller exposes:

- `POST /employees`
- `GET /employees/{id}`
- `PUT /employees/{id}`
- `DELETE /employees/{id}`

Your goal is to finish the missing SETUP/VERIFY blocks to demonstrate method safety and idempotency.

### Provided code blocks

Use these blocks to complete each test. Each block includes a brief explanation of its purpose.

**Setup: Mock repository to return an employee with `id = 1`**
*Use this when you need to simulate finding an existing employee.*

```kotlin
every {
    employeeRepository.findById(1)
} answers {
    Optional.of(Employee("Mary", "Manager", 1))
}
```

**Setup: Mock repository to return empty for `id = 2`**
*Use this when you need to simulate an employee that doesn't exist (404 case).*

```kotlin
every {
    employeeRepository.findById(2)
} answers {
    Optional.empty()
}
```

**Setup: Mock repository to return empty first, then employee**
*Use this for PUT operations where the first call creates the employee, second call updates it.*

```kotlin
every {
    employeeRepository.findById(1)
} answers {
    Optional.empty()
} andThenAnswer {
    Optional.of(Employee("Tom", "Manager", 1))
}
```

**Setup: Mock repository to return employee first, then empty**
*Use this for scenarios where an employee exists initially but gets deleted.*

```kotlin
every {
    employeeRepository.findById(1)
} answers {
    Optional.of(Employee("Tom", "Manager", 1))
} andThenAnswer {
    Optional.empty()
}
```

**Setup: Allow deleteById method to be called**
*Use this for DELETE operations. `justRun` allows void methods to be called without specifying return values.*

```kotlin
justRun {
    employeeRepository.deleteById(1)
}
```

**Setup: Mock save to return different IDs on multiple calls**
*Use this for POST operations where each call creates a new resource with a different ID.*

```kotlin
every {
    employeeRepository.save(any<Employee>())
} answers {
    Employee("Mary", "Manager", 1)
} andThenAnswer {
    Employee("Mary", "Manager", 2)
}
```

**Setup: Mock save to return employee with specified ID**
*Use this for PUT operations where the employee is updated with the same ID.*

```kotlin
every {
    employeeRepository.save(any<Employee>())
} answers {
    Employee("Tom", "Manager", 1)
}
```

**Verify: Check that save was called exactly twice**
*Use this to verify that a non-idempotent operation (like POST) was called the expected number of times.*

```kotlin
verify(exactly = 2) {
    employeeRepository.save(any<Employee>())
}
```

**Verify: Check that findById was called exactly twice**
*Use this to verify that read operations were called the expected number of times.*

```kotlin
verify(exactly = 2) {
    employeeRepository.findById(1)
}
```

**Verify: Check that no modification methods were called**
*Use this for safe operations (like GET) to ensure no side effects occurred.*

```kotlin
verify(exactly = 0) {
    employeeRepository.save(any<Employee>())
    employeeRepository.deleteById(any())
    employeeRepository.findAll()
}
```

**Verify: Check deleteById calls and no other methods**
*Use this for DELETE operations to verify the correct method was called and no others.*

```kotlin
verify(exactly = 2) {
    employeeRepository.deleteById(1)
}
verify(exactly = 0) {
    employeeRepository.save(any<Employee>())
    employeeRepository.findById(any())
    employeeRepository.findAll()
}
```

### Manual verification

1. If a method is safe, repository state modification methods must not be invoked.
2. If a method is idempotent, repeated calls either happen once or subsequent calls do not change state.
3. Run the tests:

```bash
./gradlew test
```

## How to Pass

- Your `main` branch shows completed tasks and passing tests.
- CI workflow is green.

## Insights

### Code Quality Tools: Ktlint

- Ktlint enforces Kotlin formatting according to the [Kotlin Coding Conventions](https://kotlinlang.org/docs/coding-conventions.html).
- If Ktlint modifies your source code during formatting, the build will fail.
- Run `./gradlew ktlintFormat` to format your code before building.

### Safe and Idempotent HTTP Methods

- **Safe**: do not alter server state (e.g., `GET`).
- **Idempotent**: repeated identical requests leave server in same state (`GET`, `PUT`, `DELETE`).

Understanding these helps design predictable REST APIs and the corresponding tests.


## Submission and Evaluation

### Submission Deadline

The deadline for this assignment is **October 24th**. If you fail to submit by then, you will incur a 20% penalty on your individual project score for the _URLShortener_ project. This penalty only applies if you have not attempted to submit anything of value.

### Submission Requirements

You must submit your work in **two ways**:

1. **Moodle Submission**: Upload a zip file of your complete project to Moodle
2. **GitHub Repository**: Push your changes to your GitHub repository for a potential evaluation by the teacher during the Lab 4 session

#### Moodle Zip File Submission

Create a zip file containing your complete project and upload it to Moodle. The zip file should include:

- All source code files
- Documentation files (README.md, REPORT.md)
- Test files
- Configuration files
- Any additional files you've created

#### REPORT.md File Requirements

You must create a `REPORT.md` file in your project root that includes:

1. **Description of Changes**: Detailed explanation of all modifications and enhancements you made
2. **AI Disclosure**: Complete disclosure of any AI tools or assistance used during development, including:
    - Specific AI tools used (ChatGPT, GitHub Copilot, etc.)
    - What code or documentation was generated with AI assistance
    - How much of your work was AI-assisted vs. original
    - Any AI-generated code that was modified or adapted
3. **Learning Outcomes**: What you learned from completing this assignment
4. **Technical Decisions**: Explanation of technical choices and their rationale

**Sample REPORT.md Structure**:
```markdown
# Lab 1 Git Race -- Project Report

## Description of Changes
[Detailed description of all changes made]

## Technical Decisions
[Explanation of technical choices made]

## Learning Outcomes
[What you learned from this assignment]

## AI Disclosure
### AI Tools Used
- [List specific AI tools used]

### AI-Assisted Work
- [Describe what was generated with AI assistance]
- [Percentage of AI-assisted vs. original work]
- [Any modifications made to AI-generated code]

### Original Work

- [Describe work done without AI assistance]
- [Your understanding and learning process]
```

Good luck with your assignment!