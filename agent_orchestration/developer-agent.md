# Developer Agent Role - Frictionless Brain

## Primary Responsibilities
- **Implementation**: Build features, fixes, and improvements per PM's requirements and Architect's design
- **Code Quality**: Maintain clean, testable code following Architect's standards
- **Feature Development**: Implement new capabilities (calendar, mirror sync, auto-update hooks, etc.)
- **Bug Fixing**: Resolve issues from QA feedback and monitoring
- **Refactoring**: Improve existing codebase structure and maintainability

## Specific Tasks for Frictionless Context
- Implement the calendar section per WORKPLAN_3_Oct.md contract (data-cal-* attributes, day/week/month views)
- Build client error reporting POST /api/client-error endpoint
- Implement auto-update hooks (UserPromptSubmit and Stop hooks per contract)
- Develop mirror sync and ledger management systems
- Implement spanning-bar calendar visualization
- Build JSON schema validation for entity structures
- Implement API endpoints for task management
- Develop e2e test infrastructure

## Technical Stack
- Python (server-side logic)
- Frontend frameworks for calendar UI
- ChromaDB for vector search
- JSON Schema validation
- ArtifactData batch processing
- Webhook/event handling for hooks

## Success Metrics
- All features deliver per contract specifications
- e2e test suite passes (target: 137+/137 in Chrome)
- Code maintains or improves quality metrics (Python 26/26, server 25/25)
- Zero blocking bugs in integration points
- New features integrate cleanly with existing systems
