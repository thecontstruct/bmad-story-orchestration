---
name: 'step-01-init'
description: 'Initialize adversarial auto-fix workflow by resolving file set, detecting continuation state, and creating report document'

# File References
nextStepFile: './step-02-review.md'
templateFile: '../templates/report-template.md'
continueFile: './step-01b-continue.md'
configFile: '{project-root}/_bmad/_config/custom/orchestrate/workflows/adversarial-auto-fix/config.yaml'
fileResolutionData: '../data/file-resolution.md'
---

# Step 1: Workflow Initialization

## STEP GOAL:

To initialize the adversarial auto-fix workflow by resolving the input file set, detecting continuation state, loading configuration, and creating the report document with proper state tracking.

## MANDATORY EXECUTION RULES (READ FIRST):

### Universal Rules:

- 🛑 NEVER generate content without user input
- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step with 'C', ensure entire file is read
- 📋 YOU ARE A FACILITATOR, not a content generator
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

### Role Reinforcement:

- ✅ You are an Adversarial Review Automation Specialist
- ✅ We engage in collaborative dialogue, not command-response
- ✅ You bring expertise in automated quality review, severity-aware issue detection, automatic fix delegation, and workflow state management
- ✅ User brings their domain knowledge and specific document requirements
- ✅ Together we produce something better than we could on our own

### Step-Specific Rules:

- 🎯 Focus ONLY on initialization and setup
- 🚫 FORBIDDEN to look ahead to future steps
- 💬 Handle initialization professionally and silently
- 🚪 DETECT existing workflow state and handle continuation properly

## EXECUTION PROTOCOLS:

- 🎯 Show analysis before taking any action
- 💾 Initialize report document and update frontmatter
- 📖 Set up frontmatter `stepsCompleted: ['step-01-init']` before loading next step
- 🚫 FORBIDDEN to load next step until setup is complete

## CONTEXT BOUNDARIES:

- Variables from workflow.md are available in memory
- Input path(s) provided via Skill invocation (context variable)
- Optional guidance may be provided (text or document path)
- Previous context = what's in report document + frontmatter
- Don't assume knowledge from other steps

## INITIALIZATION SEQUENCE:

### 1. Load Workflow Configuration

Load and read workflow config from `{configFile}`:

- `max_iterations` (default: 5)
- `models.review` (default: gemini-3-pro)
- `models.fix` (default: composer-1)
- `validation_task` (path to validate-adversarial-review task)
- `report_output_folder` (default: {output_folder}/validation-reports)
- `file_extensions.target` (default: [".md"])
- `file_extensions.supporting` (default: [".yaml", ".yml", ".xml", ".json"])
- `auto_fix` (default: true)

**Check for context variable overrides** (from Skill invocation):
- If `max_iterations` context variable exists: Use that value instead of config default
- If `review_model` context variable exists: Use that value instead of `models.review` from config
- If `fix_model` context variable exists: Use that value instead of `models.fix` from config
- If `auto_fix` context variable exists: Use that value instead of config default
- Otherwise: Use values from config.yaml

Set final values:
- `max_iterations_final` = context `max_iterations` OR config `max_iterations` (default: 5)
- `review_model_final` = context `review_model` OR config `models.review` (default: gemini-3-pro)
- `fix_model_final` = context `fix_model` OR config `models.fix` (default: composer-1)
- `auto_fix_final` = context `auto_fix` OR config `auto_fix` (default: true)

Resolve `report_output_folder` by replacing `{output_folder}` with the actual output folder path from BMB config.

Set context variable: `report_output_folder_path` = resolved path

### 2. Resolve Input File Set

**Load file resolution rules:** `{fileResolutionData}`

Check context for input provided via Skill invocation:
- `input_paths` context variable: Array of file paths or a single folder path

**Determine input type:**
- If `input_paths` is a single path AND that path is a directory → `input_type = "folder"`, `input_source = <folder_path>`
- If `input_paths` is an array of file paths → `input_type = "files"`, `input_source = "explicit-list"`
- If not found, prompt: "Please provide a folder path or list of file paths to review"

**Resolve file set** following rules in `{fileResolutionData}`:
- For folder input: Recursively find all files matching `file_extensions.target`
- For file list input: Validate each file exists
- Store resolved list as `file_paths` context variable
- If no target files found: Report error and halt

**Display file set summary:**
"Found {count} files to review:
{numbered list of file paths}"

### 3. Check for Existing Workflow

Check if a report document already exists:

- Generate unique workflow ID: `adversarial-{timestamp}-{random}`
- Check `{report_output_folder_path}` for existing reports with matching `file_paths` in frontmatter
- If exists, read the complete file including frontmatter
- If not exists, this is a fresh workflow

### 4. Handle Continuation (If Report Exists)

If the report exists and has frontmatter with `stepsCompleted`:

- **STOP here** and load `{continueFile}` immediately
- Do not proceed with any initialization tasks
- Let step-01b handle the continuation logic

### 5. Handle Completed Workflow

If the report exists AND all steps are marked complete in `stepsCompleted`:

- Ask user: "I found an existing adversarial review report from [date]. Would you like to:
  1. Create a new review report
  2. Continue with the existing report"
- If option 1: Generate new workflow ID and create new report
- If option 2: Load {continueFile}

### 6. Fresh Workflow Setup (If No Report)

If no report exists or no `stepsCompleted` in frontmatter:

#### A. Get Optional Guidance

Check for optional review guidance:

- Check context for `review_guidance` variable (from Skill invocation)
- If provided as text: Use directly
- If provided as document path: Load and read the guidance document
- If not provided: Set to null/empty

#### B. Generate Workflow ID

Create unique workflow ID:

- Format: `adversarial-{timestamp}-{random}`
- Example: `adversarial-20260205-abc123`
- Store in context for use in report filename

#### C. Create Report Directory

Ensure report output directory exists:

- Create directory: `{report_output_folder_path}` if it doesn't exist
- Verify write permissions

#### D. Create Initial Report Document

Copy the template from `{templateFile}` to `{report_output_folder_path}/adversarial-review-{workflow_id}.md`

Initialize frontmatter with:

```yaml
---
workflow_id: "{workflow_id}"
workflow_name: "adversarial-auto-fix"
input_type: "{input_type}"
input_source: "{input_source}"
file_paths:
  - "{file_path_1}"
  - "{file_path_2}"
review_guidance: "{review_guidance}"
stepsCompleted: ['step-01-init']
lastStep: 'step-01-init'
iteration_count: 0
max_iterations: [5]
auto_fix: true
current_status: "in-progress"
created: "{current_date}"
user_name: "{user_name}"
---
```

#### E. Initialize Report Content

Populate the report body from template, replacing:
- `{{workflow_id}}`, `{{input_source}}`, `{{file_count}}`, `{{date}}`, `{{user_name}}`
- `{{review_guidance}}` (if provided)
- `{{file_table_rows}}` with numbered table of files and their type (target)

### 7. Proceed to Review Loop

Display: **Proceeding to adversarial review loop...**

#### Menu Handling Logic:

- After setup completion, immediately load, read entire file, then execute `{nextStepFile}` to begin the review loop

---

## 🚨 SYSTEM SUCCESS/FAILURE METRICS

### ✅ SUCCESS:

- File set resolved correctly (folder recursive or explicit list)
- All target files validated
- Report document created from template (for fresh workflows)
- Frontmatter initialized with file_paths array and `stepsCompleted: ['step-01-init']`
- Optional guidance loaded (if provided)
- Workflow ID generated and stored
- Configuration loaded successfully
- Ready to proceed to step 2
- OR existing workflow properly routed to step-01b-continue.md

### ❌ SYSTEM FAILURE:

- Proceeding with step 2 without report initialization
- Not resolving file set correctly
- Not checking for existing reports properly
- Creating duplicate reports
- Not routing to step-01b-continue.md when appropriate
- Missing file validation
- Not loading configuration

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.

## CRITICAL STEP COMPLETION NOTE

ONLY WHEN initialization setup is complete and report document is created (OR continuation is properly routed), will you then immediately load, read entire file, then execute `{nextStepFile}` to begin the review loop.
