# Agent Communication Protocols - Frictionless Brain Orchestration

## Communication Channels

### 1. Primary Communication: Shared Artifacts
- **Tasks**: frictionless_brain/data/tasks.json (single source of truth for work items)
- **Decisions**: frictionless_brain/data/decisions.json (approved specifications and rationale)
- **Messages**: frictionless_brain/data/messages.json and mirror/messages.json (stakeholder communications)
- **Meetings**: frictionless_brain/data/meetings.json and mirror/meetings.json (meeting transcripts and outcomes)
- **Docs**: frictionless_brain/data/docs.json and mirror/docs.json (documentation and specifications)

### 2. Real-time Coordination: Hermes delegate_task System
- **Task Delegation**: Parent orchestrator uses `delegate_task` to assign specific work to subagents
- **Context Passing**: Each subagent receives complete context via the `context` parameter
- **Result Collection**: Subagents return structured summaries that the orchestrator synthesizes
- **Isolation**: Each subagent runs in isolated context with no access to parent's conversation history

### 3. Status Reporting: Standardized Artifacts
- **Progress Indicators**: Updated task statuses in tasks.json (pending → in_progress → completed)
- **Verification Evidence**: Test results, validation outputs, and review comments stored in relevant JSON files
- **Blocking Issues**: Escalated via special task types or direct stakeholder communication
- **Completion Signals**: Specific artifact updates or file changes indicating work completion

## Communication Patterns

### Pattern 1: Requirements Flow (PM → Architect → Developer)
1. PM extracts requirements from stakeholders and updates tasks.json with prioritized items
2. Architect reviews requirements, creates technical design, and updates relevant JSON specs
3. Developer implements based on design, referencing the task context and specifications
4. QA validates implementation against requirements and design

### Pattern 2: Feedback Loops (QA ↔ All Agents)
1. QA runs tests and identifies issues, updating tasks.json with defect information
2. Responsible agent addresses defects and marks tasks as ready for re-verification
3. QA re-tests and confirms resolution before marking task as done
4. Regression tests added to prevent reoccurrence

### Pattern 3: Release Coordination (PM ↔ DevOps ↔ All Agents)
1. PM confirms feature completeness and readiness for release
2. DevOps prepares release package and runs pre-deployment validation
3. All agents review release notes and confirm their components are included
4. DevOps executes deployment and monitors post-release health
5. PM communicates release status to stakeholders

### Pattern 4: Cross-Agent Collaboration (Ad-hoc)
1. Agents use @mentions in JSON files or direct communication for clarification
2. Shared workspace (/opt/data/frictionless_brain/) allows direct file inspection
3. Temporary coordination tasks created in tasks.json for joint efforts
4. Regular sync points via meeting transcripts and decision logs

## Message Formats and Artifacts

### Task Format (tasks.json entries)
```json
{
  "title": "Clear, actionable task description",
  "source": "Asked in Frictionless, YYYY-MM-DD or stakeholder name",
  "status": "pending|in_progress|completed|blocked",
  "assigned_to": "pm|architect|developer|qa|devops",
  "dependencies": ["task_id_1", "task_id_2"],
  "estimated_effort": "relative sizing (S/M/L/XL)",
  "acceptance_criteria": ["specific, measurable outcomes"]
}
```

### Decision Format (decisions.json entries)
```json
{
  "statement": "Clear decision statement",
  "quote": "Supporting quote from discussion",
  "date": "YYYY-MM-DD",
  "approved_by": ["stakeholder_names"],
  "related_tasks": ["task_id_1", "task_id_2"]
}
```

### Communication Best Practices
1. **Single Source of Truth**: Always reference the canonical JSON files rather than duplicating information
2. **Atomic Updates**: Make minimal, focused changes to shared artifacts
3. **Traceability**: Link tasks to decisions, decisions to meetings, meetings to source transcripts
4. **Clarity**: Use specific, actionable language in all communications
5. **Timeliness**: Update shared artifacts immediately upon completion or change
6. **Visibility**: Ensure all relevant agents can see updates to shared artifacts

## Integration with Hermes delegate_task

### Orchestration Pattern
The orchestrator (can be PM or a dedicated coordinator) uses this pattern:

1. **Task Analysis**: Break down work into delegatable sub-tasks
2. **Context Preparation**: Gather all relevant information from shared artifacts
3. **Subagent Spawning**: Use `delegate_task` with complete context for each sub-task
4. **Result Synthesis**: Collect subagent outputs and update shared artifacts
5. **Verification**: QA agent validates results before marking complete

### Delegate_task Usage Guidelines
- **Context Size**: Keep context focused but complete - include only what's needed for the task
- **Task Granularity**: Delegate tasks that can be completed independently (15-60 minutes of work)
- **Output Format**: Expect structured summaries that reference specific artifact changes
- **Error Handling**: Build retry logic for transient failures, escalate persistent issues
- **Resource Awareness**: Respect delegation limits (max 3 concurrent, depth 3)

### Example Delegation Pattern
```python
# PM Agent delegating calendar implementation
delegate_task(
    tasks=[{
        "goal": "Implement calendar section per WORKPLAN_3_Oct.md contract",
        "context": "Calendar contract requires: data-cal-* attributes, day/week/month views, spanning-bar visualization, keyboard navigation. Reference: WORKPLAN_3_Oct.md lines 11-28, 44-62. Current state: tasks E15-E18 need calendar implementation.",
        "output_schema": {
            "type": "object",
            "properties": {
                "files_modified": {"type": "array", "items": {"type": "string"}},
                "features_implemented": {"type": "array", "items": {"type": "string"}},
                "tests_passed": {"type": "object", "properties": {"e2e": {"type": "number"}, "total": {"type": "number"}}}
            },
            "required": ["files_modified", "features_implemented"]
        }
    }]
)
```

## Conflict Resolution
1. **Specification Conflicts**: Architect has final say on technical interpretation
2. **Priority Conflicts**: PM has final say on feature priority and scheduling
3. **Quality Conflicts**: QA has final say on whether work meets acceptance criteria
4. **Technical Conflicts**: Developer and Architect collaborate to find best solution
5. **Blocking Issues**: Escalate to stakeholders with clear impact assessment

## Monitoring and Improvement
- **Retrospectives**: Regular review of communication effectiveness
- **Metrics Tracking**: Measure task completion times, defect rates, rework frequency
- **Process Refinement**: Update protocols based on observed bottlenecks and pain points
- **Tool Optimization**: Refine delegate_task usage patterns for better efficiency
