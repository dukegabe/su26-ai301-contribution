# code-path-2026
AI Open Source Capstone
# Contribution [#]: Open evaluator registry button links incorrectly

**Contribution Number:** [1 / 2 / 3]  
**Student:** [Duke Gabriel]  
**Issue:** [https://github.com/Agenta-AI/agenta/issues/4535]  
**Status:** [Phase III Complete]

---

## Why I Chose This Issue

I choose this issue because it seems interesting and it seems related to UI which is familiar but still a learning curve that I'm interested in.

From reading the issue thread, I understand the current problem is that there is an invalid link or incorrect routing for the link in question. My contribution will fix the link and route the user to the expected destination.

Left a comment on the issue introducing myself — awaiting maintainer to confirm if the issue is still open. Will update this part once I hear back.

---

## Understanding the Issue

### Problem Description

The Evaluator Details Popover does not correctly handle navigation for automatic evaluators. Instead of opening the evaluator playground using the latest published revision, the component can generate navigation targets using the workflow ID. Additionally, when an automatic evaluator has no published revision, the component may still attempt to render a navigation button that relies on a null navigation target, resulting in broken navigation or runtime errors. The UI also uses an incorrect button label that references the evaluator registry, which is intended for human evaluators rather than automatic evaluators.

### Expected Behavior

When a user opens the Evaluator Details Popover:

Automatic evaluators should display an "Open evaluator playground" button.
The navigation URL should be generated using the evaluator's latest published revision ID.
If no published revision exists, the button should be disabled and provide a clear explanation to the user (for example, a tooltip indicating that no published revision is available).
Human evaluators should continue to display an "Open evaluator registry" button and navigate to the evaluator registry as they do today.

### Current Behavior

Automatic evaluators can generate navigation targets using the workflow ID instead of the latest published revision ID.
When latestRevisionId is null, the component can still attempt to access target.href, creating an invalid navigation state and potentially causing runtime errors.
Automatic evaluators display the label "Open evaluator registry", which does not accurately reflect the destination users should be taken to.
Navigation behavior may be unreliable due to the current router.push() implementation interacting with drawer close events.

### Affected Components

web/oss/src/components/SharedDrawers/TraceDrawer/components/EvaluatorDetailsPopover.tsx

---

## Reproduction Process

### Environment Setup

I set this up via the agenta Cloud versus the self hosted option, cloned the forked version onto my local machine.

### Steps to Reproduce

1. Open a trace containing evaluator results.
2. Hover over an automatic evaluator to open the Evaluator Details Popover.
3. Click the "Open evaluator registry" button.
4. Observe the generated navigation URL, opens random playground

### Reproduction Evidence

Link to my video of reproduction: https://github.com/Agenta-AI/agenta/issues/4535#issuecomment-4676006568

---

## Solution Approach

### Analysis

The issue occurs because automatic evaluators build navigation targets using workflow information instead of the latest published revision ID, and the component does not safely handle cases where latestRevisionId is null. As a result, users can be directed to incorrect URLs or encounter broken navigation when no published revision exists.

### Proposed Solution

Update the popover component to use the latest published revision ID when generating navigation targets for automatic evaluators and prevent navigation when no revision is available. The UI will also be updated to display the correct button label ("Open evaluator playground") and provide a disabled state with a tooltip when navigation is unavailable.

### Implementation Plan

Using UMPIRE framework (adapted):

**Understand:** 

The Evaluator Details Popover should provide navigation appropriate to the evaluator type.

Currently:

Automatic evaluators are intended to open the evaluator playground.
Human evaluators are intended to open the evaluator registry.

The existing implementation incorrectly handles automatic evaluators by:

Building navigation using the workflow ID instead of the latest revision ID.
Rendering navigation controls when no revision exists.
Displaying an incorrect button label ("Open evaluator registry") for automatic evaluators.

As a result, users may be sent to the wrong destination or encounter broken navigation.

**Match:** 

similar navigation pattern already exists within the evaluator navigation utilities:

buildEvaluatorTarget(...)

This helper is responsible for generating destination URLs based on evaluator metadata.

The existing codebase already distinguishes between human and automatic evaluators through:

flags.is_feedback

and

meta.is_feedback

This distinction can be reused to determine the correct navigation destination and button behavior.

**Plan:** 

1. Retrieve the Latest Published Revision

Use:

workflowLatestRevisionIdAtomFamily(...)

to obtain the most recent published revision ID for automatic evaluators.

2. Build Navigation Using Revision IDs

For automatic evaluators:

Replace the workflow ID with the latest revision ID before calling:
buildEvaluatorTarget(...)

This ensures URLs point directly to the evaluator playground revision.

3. Handle Missing Revisions Safely

If:

latestRevisionId === null

then:

Do not create a navigation target.
Render a disabled button.
Display a tooltip explaining that no published revision is available.
4. Correct User-Facing Labels

Update button text:

Human evaluator → "Open evaluator registry"
Automatic evaluator → "Open evaluator playground"
5. Simplify Navigation

Replace the existing router.push() approach with native button navigation using:

href={target.href}

This removes dependency on drawer state and avoids race conditions between navigation and drawer closing behavior.

**Implement:** 

Link to my working branch: https://github.com/dukegabe/agenta/tree/fix-issue-4535

**Review:** 

1. Review project contribution guidelines.
2. Verify linting passes.
3. Confirm TypeScript types remain valid.
4. Ensure no regressions for human evaluator behavior.
5. Verify UI consistency with existing Ant Design patterns.
6. Confirm navigation works correctly when drawers are opened and closed.

**Evaluate:** 

Manual Testing? 

One example test case is the follwing: Automatic Evaluator With Published Revision

Expected:

Button text shows:
"Open evaluator playground"
URL contains revision ID.
Navigation succeeds.

---

## Testing Strategy

Review strategy for testing Agenta AI by following this documentation here: https://agenta.ai/docs/contributing/guides/testing

More testing information is found here: https://github.com/Agenta-AI/agenta/blob/main/docs/designs/testing/README.md

Will need to get together with AI301 Build/Test Slack channel to get some help and understand which option to choose for testing but I am leaning towards "Web".

### Unit Tests

- [ ] Test case 1: [Description]
- [ ] Test case 2: [Description]
- [ ] Test case 3: [Description]

### Integration Tests

- [ ] Integration scenario 1
- [ ] Integration scenario 2

### Manual Testing

[What you tested manually and results]

---

## Implementation Notes

### Week [3] Progress

Modified:

web/oss/src/components/SharedDrawers/TraceDrawer/components/EvaluatorDetailsPopover.tsx

Key changes:

Added revision-aware navigation for automatic evaluators by retrieving the latest published revision ID from workflowLatestRevisionIdAtomFamily.

Updated navigation target generation to use the revision ID instead of the workflow ID when opening the evaluator playground.

Added null-safe handling for cases where no published revision exists.

Replaced the automatic evaluator button label from "Open evaluator registry" to "Open evaluator playground".

Added a disabled button state and tooltip when navigation is unavailable due to a missing published revision.

Simplified navigation by using the button's native href attribute instead of the existing router.push() pattern.

Design Decisions:

Chose to treat latestRevisionId === null as a non-navigable state because the atom contract indicates null represents either a loading state or the absence of a published revision.

Preserved existing behavior for human evaluators to minimize regression risk.

Used conditional rendering rather than fallback URLs to avoid sending users to incorrect destinations.

### Week [Y] Progress

[Continue documenting as you work]

### Code Changes

- **Files modified:** [List]
- **Key commits:** [Links to important commits]
- **Approach decisions:** [Why you chose certain approaches]

---

## Pull Request

**PR Link:** [GitHub PR URL when submitted]

**PR Description:** [Draft or final PR description - much of the content above can be adapted]

**Maintainer Feedback:**
- [Date]: [Summary of feedback received]
- [Date]: [How you addressed it]

**Status:** [Awaiting review / Iterating / Approved / Merged]

---

## Learnings & Reflections

### Technical Skills Gained

[What you learned technically]

### Challenges Overcome

[What was hard and how you solved it]

### What I'd Do Differently Next Time

[Reflection on your process]

---

## Resources Used

- [Link to helpful documentation]
- [Tutorial or Stack Overflow post that helped]
- [GitHub issues or discussions that helped]

