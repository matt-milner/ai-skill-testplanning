# ai-skill-testplanning
This skill contains instructions for creating a knowledge base of information about your source code and skill instructions to utilize that knowledge base for test planning.

## Introduction: Building an AI Skill for Test Identification

The goal is to create an AI skill (e.g. Copilot Agent, Claude Project, etc.) that has an in-depth knowledge of your source code and can then inform you on tests that need to be run. This is vastly superior to using your code building AI (e.g. Github Copilot or Kiro) to write your tests, because those tools will be too focused on low-level tests. This skill is built to give you a better platform to construct **system level tests** — tests that evaluate the functionality of your product from the same perspective your customers will have when they use it.

Going forward, as you make changes to the code in the form of features or bugs, the Skill has the ability to identify regression tests:

> "You changed the code over here in Feature A of the product, so here are some tests to evaluate the updated Feature A. Also, Sub-Feature B2 relies on some of the same code as Feature A, so you also need to run these tests to make sure B2 keeps working properly."

---

## Part 1: Build the Knowledge Base

Run these queries using your AI coding assistant. I strongly recommend that you start a new session to run these prompts. You should also use a **high model** to ensure you create the highest possible quality knowledge base. This will likely use quite a few tokens and possibly several hours of processing, depending on the size of the code. You might want to make changes to the prompts so that they fit better the kind of application you have created.

Each of these prompts will create **a document that you will need to save** for use in Part 2.

> Make sure that the files created by these prompts have filenames that start with **`behavioral-`** (e.g. number 1 could be named `behavioral-feature-inventory.md`).

### Behavioral prompts

1. **Feature inventory**

   ```
   Scan the codebase and produce a feature inventory document. For each distinct user-facing feature or capability, list: feature name, the entry points (API routes, UI components, CLI commands) that expose it, and a one-paragraph description of what it does from a user's perspective. Group by module/domain. Output as markdown with one section per feature.
   ```

2. **Public API contract**

   ```
   Extract all public API endpoints (REST/GraphQL/RPC). For each, document: method, path, required/optional parameters with types, validation rules enforced in code, possible response codes/shapes, and error conditions. Do not describe implementation — describe observable contract behavior only. Output as markdown grouped by resource/domain.
   ```

3. **Business/validation rules**

   ```
   Identify all business/validation rules enforced in the application layer (not database constraints) — e.g. "order total must exceed $0", "user email must be unique", "discount cannot exceed 50% without manager approval". For each rule, note: what triggers it, what the enforced condition is, and what happens on violation (error, silent rejection, fallback). Output as a markdown table:
   Rule | Trigger | Condition | Failure Behavior | Location.
   ```

4. **Roles and permissions**

   ```
   Document all user roles, permission levels, and access-control checks found in the code. For each role, list what actions/endpoints/features they can and cannot access, and where that enforcement happens.
   ```

5. **Entity lifecycles / state machines**

   ```
   Identify any entities with a defined lifecycle or state machine (e.g. Order: pending -> paid -> shipped -> delivered). Document the states, valid transitions, what triggers each transition, and any guard conditions. Output as a markdown table plus a mermaid state diagram per entity.
   ```

> Name the files in this next section **`risk-...md`**.

### Risk prompts

6. **Critical-path modules (fan-in analysis)**

   ```
   Analyze the codebase's internal dependency graph. Identify the top 20 modules/files with the highest fan-in (most depended upon by other code). For each, explain what it does and why other code relies on it. These are candidate "critical path" components.
   ```

7. **Core transaction traces**

   ```
   For each core business transaction (e.g. checkout, signup, payment processing, [insert your domain's key flows]), trace the full call path from entry point through service layers, external calls, and database writes. Document each hop, side effects (writes, emails, events published), and external dependencies invoked. Output one markdown doc per flow with a mermaid sequence diagram.
   ```

8. **External service integrations**

   ```
   List every external service integration (third-party APIs, payment gateways, message queues, cloud services). For each, note: what triggers the call, what happens on the integration failing/timing out (retry logic, fallback, error surfaced to user), and which internal features depend on it.
   ```

9. **Shared utilities / widest blast radius**

   ```
   Identify shared utility functions, base classes, or middleware used across 3+ features (e.g. auth middleware, a shared validation helper, a common data transformer). Changes here have the widest blast radius. List each, what it does, and every consuming module.
   ```

### Prompts for every future release (bug fixing / new releases)

There are two more prompts that you don't need to use to build your initial knowledge base, but that you will want to use at the end of each release so that your knowledge base is updated with the most recent changes in your code. These files should use the naming scheme **`test-...md`**:

10. **Regression-prone areas (bug-fix history)**

    ```
    Cross-reference the git commit history for messages containing "fix", "bug",  "hotfix", or linked issue numbers. Identify which files/modules appear repeatedly in bug-fix commits. Produce a ranked list — these are empirically regression-prone areas, independent of complexity metrics.
    ```

11. **High churn + high complexity files**

    ```
    Using git history, identify the files with the highest change frequency. Cross-reference against cyclomatic complexity or file size. Produce a ranked list of "high churn + high complexity" files — these are highest regression risk. Include the last 5 commit messages touching each, to hint at why they keep changing.
    ```

---

## Part 2: Create the Project

1. In your AI, create a new **Skill** or **Project**.
2. Under the **"Knowledge Base"** or **"Files"** section, upload all of the files you created in Part 1.
3. Under the **"Instructions"** section, copy and paste the contents of the **`qa-test-planning-project-instructions.md`** document.
