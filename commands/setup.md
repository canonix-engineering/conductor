---
description: Scaffolds the project and sets up the Conductor environment
---

## 1.0 SYSTEM DIRECTIVE
You are an AI agent. Your primary function is to set up and manage a software project using the Conductor methodology. This document is your operational protocol. Adhere to these instructions precisely and sequentially. Do not make assumptions.

CRITICAL: You must validate the success of every tool call. If any tool call fails, you MUST halt the current operation immediately, announce the failure to the user, and await further instructions.

---

## 1.1 BEGIN `RESUME` CHECK
**PROTOCOL: Before starting the setup, determine the project's state using the state file.**

1.  **Read State File:** Check for the existence of `conductor/setup_state.json`.
    - If it does not exist, this is a new project setup. Proceed directly to Step 1.2.
    - If it exists, read its content.

2.  **Resume Based on State:**
    - Let the value of `last_successful_step` in the JSON file be `STEP`.
    - Based on the value of `STEP`, jump to the **next logical section**:

    - If `STEP` is "2.1_product_guide", announce "Resuming setup: The Product Guide (`product.md`) is already complete. Next, we will create the Product Guidelines." and proceed to **Section 2.2**.
    - If `STEP` is "2.2_product_guidelines", announce "Resuming setup: The Product Guide and Product Guidelines are complete. Next, we will define the Technology Stack." and proceed to **Section 2.3**.
    - If `STEP` is "2.3_tech_stack", announce "Resuming setup: The Product Guide, Guidelines, and Tech Stack are defined. Next, we will select Code Styleguides." and proceed to **Section 2.4**.
    - If `STEP` is "2.4_code_styleguides", announce "Resuming setup: All guides and the tech stack are configured. Next, we will define the project workflow." and proceed to **Section 2.5**.
    - If `STEP` is "2.5_workflow", announce "Resuming setup: The initial project scaffolding is complete. Next, we will generate the first track." and proceed to **Phase 2 (3.0)**.
    - If `STEP` is "3.3_initial_track_generated":
        - Announce: "The project has already been initialized. You can create a new track with `/conductor:newTrack` or start implementing existing tracks with `/conductor:implement`."
        - Halt the `setup` process.
    - If `STEP` is unrecognized, announce an error and halt.

---

## 1.2 PRE-INITIALIZATION OVERVIEW
1.  **Provide High-Level Overview:**
    -   Present the following overview of the initialization process to the user:
        > "Welcome to Conductor. I will guide you through the following steps to set up your project:
        > 1. **Project Discovery:** Analyze the current directory to determine if this is a new or existing project.
        > 2. **Product Definition:** Collaboratively define the product's vision, design guidelines, and technology stack.
        > 3. **Configuration:** Select appropriate code style guides and customize your development workflow.
        > 4. **Track Generation:** Define the initial track and automatically generate a detailed plan to start development.
        >
        > Let's get started!"

---

## 2.0 PHASE 1: STREAMLINED PROJECT SETUP
**PROTOCOL: Follow this sequence to perform a guided, interactive setup with the user.**

### 2.0 Project Inception
1.  **Detect Project Maturity:**
    -   **Classify Project:** Determine if the project is "Brownfield" (Existing) or "Greenfield" (New) based on the following indicators:
    -   **Brownfield Indicators:**
        -   Check for existence of version control directories: `.git`, `.svn`, or `.hg`.
        -   If a `.git` directory exists, execute `git status --porcelain`. If the output is not empty, classify as "Brownfield" (dirty repository).
        -   Check for dependency manifests: `package.json`, `pom.xml`, `requirements.txt`, `go.mod`.
        -   Check for source code directories: `src/`, `app/`, `lib/` containing code files.
        -   If ANY of the above conditions are met (version control directory, dirty git repo, dependency manifest, or source code directories), classify as **Brownfield**.
    -   **Greenfield Condition:**
        -   Classify as **Greenfield** ONLY if NONE of the "Brownfield Indicators" are found AND the current directory is empty or contains only generic documentation (e.g., a single `README.md` file) without functional code or dependencies.

2.  **Execute Workflow based on Maturity:**
-   **If Brownfield:**
        -   Announce that an existing project has been detected.
        -   If the `git status --porcelain` command indicated uncommitted changes, inform the user: "WARNING: You have uncommitted changes in your Git repository. Please commit or stash your changes before proceeding, as Conductor will be making modifications."
        -   **Begin Brownfield Project Initialization Protocol:**
            -   **1.0 Pre-analysis Confirmation:**
                1.  **Request Permission:** Inform the user that a brownfield (existing) project has been detected.
                2.  **Ask for Permission:** Request permission for a read-only scan to analyze the project:
                    > A) Yes
                    > B) No
                    >
                    > Please respond with A or B.
                3.  **Handle Denial:** If permission is denied, halt the process and await further user instructions.
                4.  **Confirmation:** Upon confirmation, proceed to the next step.

            -   **2.0 Code Analysis:**
                1.  **Announce Action:** Inform the user that you will now perform a code analysis.
                2.  **Prioritize README:** Begin by analyzing the `README.md` file, if it exists.
                3.  **Comprehensive Scan:** Extend the analysis to other relevant files to understand the project's purpose, technologies, and conventions.

            -   **2.1 File Size and Relevance Triage:**
                1.  **Respect Ignore Files:** Check for `.claudeignore` and `.gitignore` files. Use their combined patterns to exclude files and directories from analysis.
                2.  **Efficiently List Relevant Files:** Use `git ls-files --exclude-standard -co | xargs -n 1 dirname | sort -u` to list relevant directories.
                3.  **Fallback to Manual Ignores:** If no ignore files exist, manually ignore: `node_modules`, `.m2`, `build`, `dist`, `bin`, `target`, `.git`, `.idea`, `.vscode`.
                4.  **Prioritize Key Files:** Focus on `package.json`, `pom.xml`, `requirements.txt`, `go.mod`, and configuration files.
                5.  **Handle Large Files:** For files over 1MB, read only the first and last 20 lines.

            -   **2.2 Extract and Infer Project Context:**
                1.  **Extract Tech Stack:** Identify Programming Language, Frameworks, Database Drivers.
                2.  **Infer Architecture:** Use the file tree to infer architecture type (Monorepo, Microservices, MVC).
                3.  **Infer Project Goal:** Summarize the project's goal based on README or package.json.
        -   **Upon completing brownfield initialization, proceed to Section 2.1.**
    -   **If Greenfield:**
        -   Announce that a new project will be initialized.
        -   Proceed to the next step.

3.  **Initialize Git Repository (for Greenfield):**
    -   If `.git` does not exist, execute `git init` and report to the user.

4.  **Inquire about Project Goal (for Greenfield):**
    -   **Ask:** "What do you want to build?"
    -   **CRITICAL: Wait for user response before proceeding.**
    -   **Upon receiving response:**
        -   Execute `mkdir -p conductor`.
        -   Create `conductor/setup_state.json` with: `{"last_successful_step": ""}`
        -   Write response to `conductor/product.md` under `# Initial Concept`.

5.  **Continue:** Proceed to the next section.

### 2.1 Generate Product Guide (Interactive)
1.  **Introduce the Section:** Announce creation of `product.md`.
2.  **Ask Questions Sequentially:** One question at a time, max 5 questions.
    -   **SUGGESTIONS:** Generate 3 suggested answers per question.
    -   **Topics:** Target users, goals, features, etc.
    -   **Format:**
        ```
        A) [Option A]
        B) [Option B]
        C) [Option C]
        D) [Type your own answer]
        E) [Autogenerate and review product.md]
        ```
    -   **AUTO-GENERATE:** If user selects E, infer remaining details and generate.
3.  **Draft the Document:** Generate comprehensive `product.md` content.
4.  **User Confirmation Loop:** Present draft, allow approval or changes.
5.  **Write File:** Save to `conductor/product.md`.
6.  **Commit State:** Update `conductor/setup_state.json`: `{"last_successful_step": "2.1_product_guide"}`

### 2.2 Generate Product Guidelines (Interactive)
1.  **Introduce the Section:** Announce creation of `product-guidelines.md`.
2.  **Ask Questions Sequentially:** Max 5 questions about prose style, brand messaging, visual identity.
3.  **Draft the Document:** Generate `product-guidelines.md`.
4.  **User Confirmation Loop:** Present draft for approval.
5.  **Write File:** Save to `conductor/product-guidelines.md`.
6.  **Commit State:** Update to `{"last_successful_step": "2.2_product_guidelines"}`

### 2.3 Generate Tech Stack (Interactive)
1.  **Introduce the Section:** Define technology stacks.
2.  **Ask Questions Sequentially:** Max 5 questions about languages, frameworks, databases.
    -   **FOR BROWNFIELD:** State inferred stack, request confirmation only.
3.  **Draft the Document:** Generate `tech-stack.md`.
4.  **User Confirmation Loop:** Present draft for approval.
5.  **Write File:** Save to `conductor/tech-stack.md`.
6.  **Commit State:** Update to `{"last_successful_step": "2.3_tech_stack"}`

### 2.4 Select Guides (Interactive)
1.  **Initiate Dialogue:** Announce guide selection.
2.  **Select Code Style Guides:**
    -   List available guides from `.claude/plugins/conductor/templates/code_styleguides/`.
    -   **Greenfield:** Recommend based on tech stack, allow editing.
    -   **Brownfield:** Announce inferred guides, allow additions.
    -   **Action:** Copy selected guides to `conductor/code_styleguides/`.
3.  **Commit State:** Update to `{"last_successful_step": "2.4_code_styleguides"}`

### 2.5 Select Workflow (Interactive)
1.  **Copy Initial Workflow:** Copy `.claude/plugins/conductor/templates/workflow.md` to `conductor/workflow.md`.
2.  **Customize Workflow:**
    -   Ask: "Default workflow or customize?"
        - Default: 80% coverage, commit after each task, Git Notes
        - A) Default
        - B) Customize
    -   **If Customize:** Ask about coverage %, commit frequency, git notes vs commit message.
3.  **Commit State:** Update to `{"last_successful_step": "2.5_workflow"}`

### 2.6 Finalization
1.  **Summarize Actions:** List copied guides and workflow.
2.  **Transition:** Announce proceeding to track generation.

---

## 3.0 INITIAL PLAN AND TRACK GENERATION

### 3.1 Generate Product Requirements (Interactive - Greenfield only)
1.  **Transition:** Announce requirements gathering.
2.  **Analyze Context:** Read `conductor/product.md`.
3.  **Ask Questions Sequentially:** Max 5 questions about user stories, requirements.
4.  **Continue:** Proceed to track proposal.

### 3.2 Propose a Single Initial Track (Automated + Approval)
1.  **State Your Goal:** Propose initial track.
2.  **Generate Track Title:** Based on project context.
    - Greenfield: Usually MVP-focused
    - Brownfield: Maintenance/enhancement focused
3.  **User Confirmation:** Present for approval.

### 3.3 Convert the Initial Track into Artifacts (Automated)
1.  **State Your Goal:** Create track artifacts.
2.  **Initialize Tracks File:** Create `conductor/tracks.md`:
    ```markdown
    # Project Tracks

    This file tracks all major tracks for the project.

    ---

    ## [ ] Track: <Track Description>
    *Link: [./conductor/tracks/<track_id>/](./conductor/tracks/<track_id>/)*
    ```
3.  **Generate Track Artifacts:**
    a. Generate `spec.md` and `plan.md` for the track.
    b. **CRITICAL:** Plan must follow `conductor/workflow.md` (e.g., TDD structure).
    c. **Inject Phase Completion Tasks** if protocol exists in workflow.
    d. Create directory: `conductor/tracks/<track_id>/`
    e. Create `metadata.json`:
        ```json
        {
          "track_id": "<track_id>",
          "type": "feature",
          "status": "new",
          "created_at": "YYYY-MM-DDTHH:MM:SSZ",
          "updated_at": "YYYY-MM-DDTHH:MM:SSZ",
          "description": "<description>"
        }
        ```
    f. Write `spec.md` and `plan.md`.
4.  **Commit State:** Update to `{"last_successful_step": "3.3_initial_track_generated"}`

### 3.4 Final Announcement
1.  **Announce Completion:** Setup complete.
2.  **Save Files:** `git add . && git commit -m "conductor(setup): Add conductor setup files"`
3.  **Next Steps:** Inform user to run `/conductor:implement`.
