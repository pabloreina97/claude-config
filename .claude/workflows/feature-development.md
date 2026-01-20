# Feature Development Workflow

This document describes the coordinated three-agent workflow for developing new features in this Next.js application.

## Overview

The workflow involves three specialized agents working in sequence, coordinated by the main orchestrator:

```
User Request
    ↓
[1. Requirements Analyst] → Gathers requirements, creates spec document
    ↓
[2. Architecture Expert] → Designs technical architecture and implementation plan
    ↓
[3. Frontend Developer] → Implements code following the plan
    ↓
[Main Orchestrator] → Pushes to GitHub, monitors Vercel deployment
    ↓
Success? No → Agents fix issues → Retry
    ↓
Success? Yes → Feature deployed! ✓
```

## The Agents

### 1. Requirements Analyst
**File**: `.claude/agents/requirements-analyst.md`

**Purpose**: Gather complete requirements through interactive questioning

**Process**:
- Asks clarifying questions about functionality, UI/UX, data, and technical needs
- Identifies edge cases and constraints
- Documents everything in a structured format

**Output**: Requirements document at `.claude/docs/features/<feature-name>-01.md`

**Example Usage**:
```
User: "I want to add a user dashboard"
↓
Analyst asks: What should be displayed? Who can access it? What data is needed?
↓
Creates: .claude/docs/features/user-dashboard-01.md
```

### 2. Next.js Architecture Expert
**File**: `.claude/agents/nextjs-architecture-expert.md`

**Purpose**: Design technical architecture based on requirements

**Process**:
- Reads requirements document
- Analyzes current codebase
- Decides on routes, components, data fetching strategy
- Creates detailed implementation plan

**Output**: Architectural specifications and implementation roadmap in `.claude/docs/features/<feature-name>-02.md`

**Example Output**:
```markdown
## Implementation Plan

### Routes
- /dashboard - Server Component (direct DB access)
- /dashboard/settings - Nested route

### Components
1. DashboardCard.tsx (shared, reusable)
2. DashboardStats.tsx (client component for charts)
3. page.tsx (server component for layout)

### Implementation Order
1. Create shared DashboardCard
2. Create main dashboard page
3. Add stats component with interactivity
```

### 3. Frontend Developer
**File**: `.claude/agents/frontend-developer.md`

**Purpose**: Implement the code following architectural specifications

**Process**:
- Receives implementation plan
- Writes components, pages, and routes
- Makes atomic commits for each logical unit
- Tests as they go

**Output**: Working code with clean git history

**Example Commits**:
```
feat: Add DashboardCard shared component
feat: Add dashboard page with server-side data fetching
feat: Add interactive stats chart component
style: Make dashboard responsive on mobile
```

### 4. Main Orchestrator (Root Agent)
**Purpose**: Coordinate the workflow and manage deployment

**Process**:
- Invokes agents in sequence
- When changes are ready, run `npm run dev`
- Handles errors. Directs agents to fix issues if needed.
- If there are no errors. Pushes to GitHub and monitors Vercel deployment via MCP

**Output**: Deployed feature in production

## Workflow Steps

### Step 1: User Requests Feature

User describes what they want to build:
```
User: "I want to add a blog feature where users can write and publish posts"
```

### Step 2: Requirements Gathering

The orchestrator invokes the **Requirements Analyst**:
```
Orchestrator: "Requirements Analyst, please gather requirements for this blog feature"
```

The analyst:
1. Asks clarifying questions interactively
2. Documents all requirements
3. Creates `.claude/docs/features/blog-feature-01.md`
4. Reports completion

### Step 3: Architecture Design

The orchestrator invokes the **Architecture Expert**:
```
Orchestrator: "Architecture Expert, read .claude/docs/features/blog-feature-01.md and design the implementation"
```

The architect:
1. Reads the requirements document
2. Analyzes current codebase structure
3. Makes architectural decisions
4. Creates detailed implementation plan
5. Communicates plan to Frontend Developer

### Step 4: Implementation

The orchestrator invokes the **Frontend Developer**:
```
Orchestrator: "Frontend Developer, implement the blog feature following the Architecture Expert's plan"
```

The developer:
1. Follows implementation order from the plan
2. Writes components and pages
3. Makes atomic commits after each logical change
4. Tests functionality
5. Reports completion

### Step 5: Start local server and deployment

The **Orchestrator**:

1. Run `npm run dev`.

1. If everything is OK, pushes all commits to GitHub
   ```bash
   git push origin main
   ```

2. Monitors Vercel deployment using MCP
   ```
   mcp__vercel-awesome-ai__list_deployments
   mcp__vercel-awesome-ai__get_deployment_build_logs
   ```

3. Checks deployment status

### Step 6: Error Handling (if needed)

If the local run dev fails:

1. **Orchestrator** analyzes build logs
2. Identifies the error (TypeScript, build error, etc.)
3. Invokes appropriate agent to fix:
   - Architecture issues → **Architecture Expert**
   - Implementation bugs → **Frontend Developer**

4. Agent fixes the issue and commits
5. **Orchestrator** refresh the local server
6. Repeat until succeeds and push to GitHub

## Usage Examples

### Example 1: Simple Feature

```
User: "Add a contact form to the site"

Requirements Analyst:
- What fields? (name, email, message)
- Where to send submissions? (database, email)
- Creates: .claude/docs/contact-form.md

Architecture Expert:
- Decides: /contact page, Server Action for submission
- Creates implementation plan
- Tells developer: "Create /contact page with Server Component and form submission"

Frontend Developer:
- Creates src/app/contact/page.tsx
- Adds form with validation
- Implements server action
- Commits: "feat: Add contact form with server-side submission"

Orchestrator:
- Start local server. No errors.
- Pushes to GitHub
- Monitors Vercel deployment
- ✓ Success
```

### Example 2: Complex Feature with Iteration

```
User: "Add a full blog system"

Requirements Analyst:
- Asks about: post creation, editing, markdown support, categories, etc.
- Creates: .claude/docs/blog-system.md (detailed spec)

Architecture Expert:
- Decides:
  * Database: Post model with categories
  * Routes: /blog, /blog/[slug], /blog/create
  * Components: PostCard, PostEditor, MarkdownRenderer
- Creates detailed implementation plan

Frontend Developer:
- Updates Prisma schema
- Creates components in order
- Makes multiple commits
- Commits example:
  * "feat: Add Post model to database schema"
  * "feat: Add blog post listing page"
  * "feat: Add markdown post editor"
  * "feat: Add post detail page with SSG"

Orchestrator:
- Starts local server. ❌ Build error: TypeScript error in PostEditor.
- Asks Frontend Developer to fix
- Developer fixes, commits
- Refresh local server. No errors.
- Pushes to GitHub
- Monitors deployment
- ✓ Success
```

## Key Principles

### 1. Separation of Concerns
- **Analyst** = What to build (requirements)
- **Architect** = How to build it (technical design)
- **Developer** = Build it (implementation)
- **Orchestrator** = Deploy it (deployment)

### 2. Documentation First
Always create requirements doc before coding. This ensures:
- Clear understanding of scope
- Proper architectural decisions
- Implementation follows a plan

### 3. Atomic Commits
Small, focused commits make it easy to:
- Review changes
- Debug issues
- Rollback if needed

### 4. Test Before Deploy
- Developer tests during implementation
- Orchestrator verifies via npm run dev and then in Vercel deployment.
- Fix and retry until success

### 5. Iterative Refinement
If the code fails:
- Don't panic, analyze the error
- Fix systematically
- Commit the fix
- Deploy again

## File Locations

```
.claude/
├── agents/
│   ├── requirements-analyst.md      # Step 1: Requirements gathering
│   ├── nextjs-architecture-expert.md # Step 2: Architecture design
│   └── frontend-developer.md         # Step 3: Implementation
├── docs/
│   ├── _template.md                  # Template for requirements docs
│   └── features/
│       └── <feature-name>-01.md      # Feature requirements (created by analyst)
│       └── <feature-name>-02.md      # Implementation plan (created by architect)
└── workflows/
    └── feature-development.md        # This file
```

## How to Use This Workflow

### As a User

Simply describe what you want:
```
"I want to add [feature description]"
```

The orchestrator will handle invoking the agents in the right order.

### As the Orchestrator

1. Invoke Requirements Analyst:
   ```
   @requirements-analyst Please gather requirements for [feature]
   ```

2. Wait for requirements doc to be created

3. Invoke Architecture Expert:
   ```
   @nextjs-architecture-expert Read .claude/docs/[feature].md and create implementation plan
   ```

4. Wait for architectural plan

5. Invoke Frontend Developer:
   ```
   @frontend-developer Implement [feature] following the plan from Architecture Expert
   ```

6. When implementation is complete:
   - Run `npm run dev`
   - Check for errors
   - If errors, direct appropriate agent to fix
   - Repeat until success and push to GitHub.

### Handling Errors

**TypeScript/Build Errors**:
- Frontend Developer fixes implementation

**Architectural Issues**:
- Architecture Expert revises plan
- Frontend Developer re-implements

**Deployment Configuration**:
- Check Vercel settings
- Update environment variables if needed

## Benefits

1. **Clarity**: Each agent has a clear role
2. **Quality**: Requirements documented before coding
3. **Maintainability**: Atomic commits, clean history
4. **Reliability**: Automated deployment verification
5. **Efficiency**: Agents work in parallel when possible
6. **Learning**: Clear documentation of decisions

## Future Enhancements

Potential improvements to the workflow:

- Add testing agent for automated tests
- Add review agent for code quality checks
- Add documentation agent for updating docs
- Add performance agent for optimization
- Integrate with CI/CD for automated testing

---

**Last Updated**: 2025-10-28
**Version**: 1.1.0
