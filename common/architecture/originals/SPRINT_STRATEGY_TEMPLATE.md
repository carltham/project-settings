# Sprint Planning Strategy - Generic Template

**Project:** [PROJECT_NAME]  
**Duration:** [TOTAL_WEEKS] weeks ([NUM_SPRINTS] sprints)  
**Team:** [TEAM_SIZE] members  
**Total Capacity:** [TOTAL_STORY_POINTS] story points

---

## Sprint Cycle

### Sprint Duration
- **[SPRINT_DURATION] weeks (Weeks [WEEK_START]-[WEEK_END]):** Development work
- **[REVIEW_DAY] [REVIEW_TIME]:** Sprint review + retrospective
- **[PLANNING_DAY] [PLANNING_TIME]:** Next sprint kickoff

### Weekly Schedule

**[PLANNING_DAY] [PLANNING_TIME] ([PLANNING_DURATION])**
- Sprint planning / review previous metrics
- Confirm priorities and risks
- Assign tasks to developers

**Daily [STANDUP_TIME] ([STANDUP_DURATION])**
- Standup: yesterday, today, blockers
- Post in [COMMUNICATION_CHANNEL]
- Escalate blockers immediately

**[MIDPOINT_DAY] (Midpoint Check)**
- Velocity confirmation
- Adjust if needed

**[REVIEW_DAY] [REVIEW_TIME] ([REVIEW_DURATION])**
- Sprint review: demo completed work
- Gather stakeholder feedback

**[RETRO_DAY] [RETRO_TIME] ([RETRO_DURATION])**
- Retrospective: what went well, what to improve
- Action items for next sprint

---

## Task Management

### Starting a Task
1. Pick task from active/ folder
2. Create feature branch: `feature/TASK-id-name`
3. Update task file: status = 🟡 In Progress
4. Post in [COMMUNICATION_CHANNEL]: "Starting TASK-id"

### Daily Progress
- Commit work to feature branch
- Update % progress in task file
- Post standup in [COMMUNICATION_CHANNEL]

### Completing a Task
1. Ensure all acceptance criteria met
2. Unit tests >80% coverage
3. Create PR linked to task file
4. Get [NUM_REVIEWERS]+ code reviews
5. Merge to main
6. Move file: `active/TASK-*.md` → `done/TASK-*.md`
7. Update STATUS.md files
8. Announce in [COMMUNICATION_CHANNEL]: "Completed TASK-id"

---

## Success Metrics

| Metric | Target | Check |
|--------|--------|-------|
| Velocity | [VELOCITY_TARGET] pts/sprint | Every [REVIEW_DAY] |
| Code Coverage | >[COVERAGE_TARGET]% | Before merge |
| Tests Passing | >[TEST_TARGET]% | PR requirement |
| Security | [SECURITY_REQUIREMENT] | Weekly |
| Code Review Time | <[REVIEW_TIME_LIMIT] hrs | Monitor daily |
| On-Time Delivery | [DELIVERY_TARGET]% | End of sprint |

---

## Risk Management

### Red Flags (Stop & Discuss)
- 🚩 Task blocked >[BLOCKER_THRESHOLD] days
- 🚩 Coverage drops <[COVERAGE_THRESHOLD]%
- 🚩 [SECURITY_ISSUE_TYPE] vulnerability found
- 🚩 Team velocity drops [VELOCITY_DROP]%+
- 🚩 Dependency not met on time

### Mitigation Actions
- Daily standups catch issues early
- Code review before merge prevents bugs
- Pair programming unblocks technical debt
- [DOMAIN] clarity prevents rework

---

## Dependencies

### Between Sprints
1. **Sprint 1 → 2:** [PHASE_1] complete = [PHASE_2] unblocked
2. **Sprint 2 → 3:** [PHASE_2] complete = [PHASE_3] unblocked
3. **[ADDITIONAL_DEPENDENCIES]**

### Within Sprints
- Critical path identified in each sprint plan
- Blocking tasks marked clearly
- Dependent tasks scheduled after blockers

---

## Sprint Capacity Planning

### Allocation by Role

| Role | [SPRINT_1] | [SPRINT_2] | [SPRINT_3] | ... |
|------|-----------|-----------|-----------|-----|
| [ROLE_1] | [PTS] | [PTS] | [PTS] | ... |
| [ROLE_2] | [PTS] | [PTS] | [PTS] | ... |
| [ROLE_3] | [PTS] | [PTS] | [PTS] | ... |
| [ADDITIONAL_ROLES] | ... | ... | ... | ... |

### Capacity Buffer
- [BUFFER_PERCENT]% buffer built in
- Handles meetings, reviews, interruptions
- Allows learning/spike time

---

## Blockers & Escalation

### Escalation Path
1. **Developer → Team Lead** (same day)
2. **Team Lead → Sprint Owner** (within [ESCALATION_1] hours)
3. **Sprint Owner → Project Lead** (within [ESCALATION_2] hours)

### Blocker Resolution
- Pair programming for technical blockers
- [DOMAIN] review for design issues
- External resources for tool/vendor issues
- Pivot to dependent work while blocked

---

## Quality Gates

### Before Merging to Main
- ✅ Unit tests passing (>[COVERAGE_TARGET]% coverage)
- ✅ Code review approved ([NUM_REVIEWERS]+ reviewers)
- ✅ No security vulnerabilities
- ✅ No performance regressions
- ✅ Acceptance criteria met

### End of Sprint Criteria
- ✅ All tasks moved to done/ or blocked with plan
- ✅ Velocity >= [VELOCITY_TARGET]% of target
- ✅ No critical issues in done tasks
- ✅ Documentation updated
- ✅ Next sprint dependencies unblocked

---

## Communication

### Daily
- [STANDUP_TIME] standup ([STANDUP_DURATION])
- [COMMUNICATION_CHANNEL] updates

### Weekly
- [PLANNING_DAY]: Planning ([PLANNING_DURATION])
- [REVIEW_DAY]: Review + Retro ([REVIEW_DURATION] + [RETRO_DURATION])
- [MIDPOINT_DAY]: Midpoint check ([MIDPOINT_DURATION])

### On Demand
- Blockers posted immediately
- [DOMAIN] decisions in [DECISION_CHANNEL]
- Security issues escalated to lead

---

## Retrospective Template

**What went well?**
- 3-5 items team did right

**What needs improvement?**
- 2-3 areas to focus on

**Action items for next sprint**
- Specific, measurable changes
- Owner assigned for each

---

## Milestone Tracking

### Sprint 1 ([START_DATE] - [END_DATE])
✓ [SPRINT_1_DELIVERABLES]
→ Unblocks: [UNBLOCKED_PHASES]

### Sprint 2 ([START_DATE] - [END_DATE])
✓ [SPRINT_2_DELIVERABLES]
→ Unblocks: [UNBLOCKED_PHASES]

### [ADDITIONAL_SPRINTS]
✓ [SPRINT_DELIVERABLES]
→ Unblocks: [UNBLOCKED_PHASES]

### Final Milestone ([DATE])
✓ [FINAL_DELIVERABLES]
→ Ready for [NEXT_PHASE]

---

## Instructions for Customization

1. Replace all `[PLACEHOLDER]` values with project-specific data
2. Customize weekly schedule based on team timezone
3. Set success metrics based on project quality standards
4. Identify domain-specific risks and mitigation
5. Add additional roles/sprints as needed
6. Set escalation thresholds based on org structure
7. Define communication channels (Slack, Teams, etc.)
8. Adjust capacity buffer based on org interruption rate

---

**Status:** Template - Ready to customize  
**Version:** 1.0  
**Last Updated:** 2026-07-12

