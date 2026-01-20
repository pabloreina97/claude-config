---
description: Start a new session with context management and sub-agent workflow
---

# Session workflow

You are starting a new work session. Follow these rules strictly:

## Context management

1. **Before starting any work**: Read the `.claude/context/context_session_{{SESSION_ID}}.md` file to get full context
   - If the file doesn't exist, create it with an initial structure including:
     - Session ID and date
     - Overall plan/goals
     - Work completed
     - Next steps
   - Ask the user for the session ID if not provided (suggest using a number or descriptive name)

2. **During work**: Continuously update the context file as you progress

3. **After finishing work**: Update the `.claude/context/context_session_{{SESSION_ID}}.md` file with:
   - What was accomplished
   - Any decisions made
   - Files created/modified
   - Next steps or pending tasks

## Sub-agent workflow

**Discovering available sub-agents**:
1. Check the `.claude/agents/` directory to see which sub-agents are available
2. Review each agent's description and capabilities
3. Based on the user's task, select ONLY the sub-agents that are relevant and necessary

**Important sub-agent rules**:
- When passing tasks to sub-agents, ALWAYS include the context file path (`.claude/context/context_session_{{SESSION_ID}}.md`)
- Sub-agents will do research and create documentation, but YOU will do the actual implementation
- After each sub-agent finishes, read the related documentation they created to get full context before executing the plan
- Only consult sub-agents that are truly relevant to the current task (don't consult all agents by default)

## Your workflow

1. **Check if the user provided instructions**: If the user included a task description or instructions in their message, proceed directly with that task. Otherwise, ask what they want to work on.
2. **Ask for session ID if needed**: If not already provided, ask the user for the session ID (suggest using a number or descriptive name)
3. Read or create the context file for this session
4. Understand the user's request
5. Check `.claude/agents/` directory to discover available sub-agents
6. Identify which sub-agents (if any) are relevant for the current task
7. Consult only the relevant sub-agents, passing them the context file path
8. Read the documentation created by sub-agents
9. Execute the implementation yourself
10. Update the context file with progress and outcomes

**Note**: The user can either:
- Provide task instructions in the same message: `/session <instructions>`
- Or wait to be asked what they want to work on
