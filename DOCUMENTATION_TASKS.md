# MSP Checkmate Documentation Tasks

## Documentation Standards
Following claude.md guidelines:
- Second-person voice ("you")
- MDX format with YAML frontmatter
- Test all code examples
- Include prerequisites
- Relative paths for internal links

## Module Documentation Tasks

### 1. Company Overview Module
**Path:** `/modules/company-overview.mdx`
- [ ] Review `D:\git\checkmate-main\app\(auth)\company_overview\` implementation
- [ ] Document dashboard components (SummaryCards, ComplianceIssuesCard, RecentRunsCard, PlannerItemsCard)
- [ ] Document audit functionality from `/company_overview/audit/`
- [ ] Include filtering and company selection features
- [ ] Add screenshots of dashboard views
- [ ] Create user guide: `/user-guides/company-overview-guide.mdx`

### 2. Surveys Module
**Path:** `/modules/surveys.mdx`
- [ ] Review `D:\git\checkmate-main\app\(auth)\surveys\` implementation
- [ ] Document survey types (NPS, General surveys)
- [ ] Document survey creation and templates
- [ ] Document analytics features (`/surveys/analytics/`)
- [ ] Document person-level responses and word maps
- [ ] Document survey groups functionality
- [ ] Include email template configuration
- [ ] Create user guide: `/user-guides/surveys-guide.mdx`

### 3. Goals Module
**Path:** `/modules/goals.mdx`
- [ ] Review `D:\git\checkmate-main\app\(auth)\goals\` implementation
- [ ] Document goal creation from survey responses
- [ ] Document goal tracking and progress monitoring
- [ ] Document goal-planner integration
- [ ] Include goal prioritization features
- [ ] Create user guide: `/user-guides/goals-guide.mdx`

### 4. Budgets Module
**Path:** `/modules/budgets.mdx`
- [ ] Review `D:\git\checkmate-main\app\(auth)\budgets\` implementation
- [ ] Document budget creation and editing
- [ ] Document products and templates
- [ ] Document company-specific budgets
- [ ] Include budget tracking features
- [ ] Create user guide: `/user-guides/budgets-guide.mdx`

### 5. Calendar Module
**Path:** `/modules/calendar.mdx`
- [ ] Review `D:\git\checkmate-main\app\(auth)\calendar\` implementation
- [ ] Document calendar views (list, me, recurring)
- [ ] Document calendar settings
- [ ] Document Microsoft 365 calendar sync
- [ ] Include scheduling features
- [ ] Create user guide: `/user-guides/calendar-guide.mdx`

### 6. Microsoft 365 Module
**Path:** `/modules/microsoft-365.mdx`
- [ ] Review `D:\git\checkmate-main\app\(auth)\m365\` implementation
- [ ] Document Secure Score functionality
- [ ] Document improvement actions and controls
- [ ] Document tenant access configuration
- [ ] Include integration setup steps
- [ ] Create user guide: `/user-guides/microsoft-365-guide.mdx`

### 7. Ticketing Module
**Path:** `/modules/ticketing.mdx`
- [ ] Review `D:\git\checkmate-main\app\(auth)\tickets\` implementation
- [ ] Document ticket creation (including AI-powered creation)
- [ ] Document ticket templates
- [ ] Document PSA integration (ConnectWise, Halo)
- [ ] Document ticket categories and settings
- [ ] Create user guide: `/user-guides/ticketing-guide.mdx`

### 8. Compliance Module
**Path:** `/modules/compliance.mdx`
- [ ] Review `D:\git\checkmate-main\app\(auth)\compliance\` implementation
- [ ] Document compliance frameworks and checks
- [ ] Document compliance runs and scheduling
- [ ] Document compliance history tracking
- [ ] Document compliance settings and groups
- [ ] Include framework matrix features
- [ ] Create user guide: `/user-guides/compliance-guide.mdx`

### 9. Devices Module
**Path:** `/modules/devices.mdx`
- [ ] Review `D:\git\checkmate-main\app\(auth)\devices\` implementation
- [ ] Document device tracking and management
- [ ] Document device types configuration
- [ ] Document device settings
- [ ] Include lifecycle management features
- [ ] Create user guide: `/user-guides/devices-guide.mdx`

### 10. Reporting Module
**Path:** `/modules/reporting.mdx`
- [ ] Review `D:\git\checkmate-main\app\(auth)\reporting\` implementation
- [ ] Document report builder functionality
- [ ] Document report templates
- [ ] Document report sharing features
- [ ] Document PDF export capabilities
- [ ] Document module availability settings
- [ ] Create user guide: `/user-guides/reporting-guide.mdx`

## User Guide Structure

### User Guide Template
Each user guide should include:
1. **Getting Started** - First-time setup and orientation
2. **Common Tasks** - Step-by-step procedures for frequent operations
3. **Advanced Features** - Power user functionality
4. **Troubleshooting** - Common issues and solutions
5. **Best Practices** - Recommendations for optimal use
6. **FAQs** - Frequently asked questions

### User Guide Directory Structure
```
/user-guides/
├── index.mdx                    # User guides overview and navigation
├── quick-start.mdx              # Platform quick start guide
├── company-overview-guide.mdx   # Company Overview module guide
├── surveys-guide.mdx            # Surveys module guide
├── goals-guide.mdx              # Goals module guide
├── budgets-guide.mdx            # Budgets module guide
├── calendar-guide.mdx           # Calendar module guide
├── microsoft-365-guide.mdx      # Microsoft 365 module guide
├── ticketing-guide.mdx          # Ticketing module guide
├── compliance-guide.mdx         # Compliance module guide
├── devices-guide.mdx            # Devices module guide
├── reporting-guide.mdx          # Reporting module guide
├── settings-guide.mdx           # Settings configuration guide
├── integrations-guide.mdx       # Integrations setup guide
└── admin-guide.mdx              # Administrator guide
```

## Additional Documentation Tasks

### Settings Documentation
- [ ] Create comprehensive settings guide covering all settings areas
- [ ] Document each settings section with screenshots
- [ ] Include configuration best practices

### Integration Documentation
- [ ] Document ConnectWise integration setup and configuration
- [ ] Document Halo PSA integration setup and configuration
- [ ] Document Microsoft 365 integration setup
- [ ] Document API access and webhook configuration

### Admin Documentation
- [ ] Document admin portal features
- [ ] Document user management procedures
- [ ] Document role and permission configuration
- [ ] Document tenant management

## Navigation Updates Required

### docs.json Updates
- [ ] Add "Modules" section with all module documentation
- [ ] Add "User Guides" section with all user guides
- [ ] Update existing navigation structure to include new content
- [ ] Ensure proper hierarchy and grouping

## Content Guidelines

### For Each Module Documentation:
1. **Review actual codebase** - Never make assumptions about features
2. **Include real paths** - Reference actual file locations when relevant
3. **Test procedures** - Verify all step-by-step instructions work
4. **Add visuals** - Include diagrams or screenshots where helpful
5. **Cross-reference** - Link to related modules and guides

### For Each User Guide:
1. **Task-oriented** - Focus on what users want to accomplish
2. **Progressive disclosure** - Start simple, add complexity gradually
3. **Real scenarios** - Use realistic MSP use cases
4. **Visual aids** - Include annotated screenshots for complex procedures
5. **Searchable** - Use clear headings and keywords

## Priority Order

### Phase 1 - Core Modules (Immediate)
1. Company Overview
2. Compliance
3. Ticketing
4. Reporting

### Phase 2 - Planning & Engagement (Next)
1. Planner (update existing)
2. Goals
3. Surveys
4. Calendar

### Phase 3 - Supporting Modules (Following)
1. Budgets
2. Devices
3. Microsoft 365
4. Settings (enhance existing)

### Phase 4 - Guides & Advanced (Final)
1. User guides for all modules
2. Integration guides
3. Admin documentation
4. Advanced configurations

## Quality Checklist

For each documentation piece:
- [ ] Frontmatter includes title and description
- [ ] Second-person voice used throughout
- [ ] Code examples tested and working
- [ ] Internal links use relative paths
- [ ] Prerequisites clearly stated
- [ ] Follows existing documentation patterns
- [ ] No assumptions or guessed information
- [ ] Based on actual codebase review

## Notes

- Always check `D:\git\checkmate-main\` for actual implementation
- Never document features that don't exist
- Ask for clarification when functionality is unclear
- Test all procedures in the actual application when possible
- Maintain consistency with existing documentation style