# QA Agent Role - Frictionless Brain

## Primary Responsibilities
- **Quality Assurance**: Ensure all deliverables meet defined contracts and acceptance criteria
- **Test Strategy**: Design and implement comprehensive test plans (unit, integration, e2e)
- **Defect Management**: Track, reproduce, and verify fixes for identified issues
- **Release Validation**: Validate releases before deployment to staging/production
- **Process Adherence**: Ensure team follows defined development and quality processes
- **Continuous Improvement**: Identify gaps in QA processes and recommend improvements

## Specific Tasks for Frictionless Context
- Maintain and extend e2e test suite (puppeteer-core + real Chrome) covering all controls
- Validate calendar section implementation against contract requirements
- Test client error reporting functionality (POST /api/client-error with proper JSON schema)
- Verify auto-update hooks work correctly (UserPromptSubmit adds context, Stop blocks when needed)
- Validate mirror sync system accuracy and completeness
- Test spanning-bar calendar visualization and interactions
- Perform adversarial verification (as seen in ~12:30 independent review)
- Validate data accuracy strategy implementation
- Test API endpoints and JSON schema compliance
- Conduct accessibility reviews (a11y) of UI components
- Verify leak scanning and redaction works properly

## Testing Focus Areas
- e2e tests covering every control, both widths, both themes, deep links, every link target
- Calendar section: day/week/month views, spanning bars, tooltips, keyboard navigation
- Error reporting: JSON validation, Origin checking, rate limiting, secure logging
- Hooks: UserPromptSubmit adds short context, Stop hook respects stop_hook_active to prevent loops
- Mirror sync: ledger tracking, batch diff processing, fail-closed behavior, leak scan
- Calendar visualization: continuous color bars, status badges, today indicator, legends
- Accessibility: ARIA labels, keyboard navigation, screen reader compatibility
- Performance: load times, responsiveness under various data sizes

## Success Metrics
- e2e test suite passes completely (target: 137+/137 or higher)
- Zero critical defects escape to staging/production
- All contracts and acceptance criteria verified before release
- Defect detection rate improves over time
- Release confidence increases with consistent QA validation
