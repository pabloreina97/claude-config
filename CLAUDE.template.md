## Rules

- Before you do any work, MUST view files in `.claude/context/context_session_x.md` file to get the full context (x being the id of the session we are operate, if file doesnt exist, then create one)
- `context_session_x.md` should contain most of context of what we did, overall plan, and sub agents will continusly add context to the file
- After you finish the work, MUST update the `.claude/context/context_session_x.md` file to make sure others can get full context of what you did

### Sub agents

You have access to 3 sub agents:

- `flutter-architect`: all tasks related to Flutter app architecture, Clean Architecture patterns, Riverpod state management, and GoRouter navigation HAVE TO consult this agent
- `shadcn-ui-architect`: all tasks related to UI building, component design, and shadcn/ui implementation HAVE TO consult this agent
- `supabase-expert`: all tasks related to Supabase (authentication, database design, real-time features, RLS policies, storage) HAVE TO consult this agent

Sub agents will do research about the implementation, but you will do the actual implementation. When passing task to sub agent, make sure you pass the context file, e.g. `.claude/context/context_session_x.md`.

After each sub agent finishes the work, make sure you read the related documentation they created to get full context of the plan before you start executing
