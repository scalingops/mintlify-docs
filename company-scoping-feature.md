# Company Scoping Feature

**Date:** 2025-09-15
**Version:** 1.1
**Implementation:** Complete
**Last Updated:** 2025-09-15 (Consistency Fixes)

## Overview

The Company Scoping feature provides a flexible data filtering system that allows users to view and interact with data across different company contexts. This feature enables three distinct scoping modes: Global View, Single Company View, and Assigned Companies View.

## Feature Description

### Scoping Modes

#### 1. Global View
- **Description**: Shows data from all companies within the tenant
- **Use Case**: Administrative oversight, tenant-wide reporting and analytics
- **Access**: Available to admin users with appropriate permissions
- **Icon**: Globe icon in the company selector

#### 2. Single Company View
- **Description**: Filters all data to a specific selected company
- **Use Case**: Focused work on individual company data, client-specific operations
- **Access**: Available to all users with company access permissions
- **Selection**: Choose from dropdown list of available companies

#### 3. Assigned Companies View
- **Description**: Shows aggregated data from companies assigned to the current user
- **Use Case**: Account managers or technicians responsible for specific company sets
- **Access**: Available to users with assigned companies
- **Display**: Shows count of assigned companies in selector

### Company User Restrictions

- **Company Users**: Users with `user_type = "company"` are automatically restricted to their specific company
- **Impersonation**: When impersonating a company user, the system locks to that user's company
- **UI Behavior**: Company selector is hidden entirely for company users (real or impersonated)

## Technical Implementation

### Core Components

#### 1. Company Scope Types (`types/company.ts`)
```typescript
export type CompanyScope = "global" | "single" | "assigned";

export type CompanyScopeFilter = {
  scope: CompanyScope;
  companyId?: string | null;
  companyIds?: string[];
};
```

#### 2. Server Action (`app/actions/setCompanyScope.ts`)
Manages company scope cookies and synchronizes state:
- Sets `company_scope` cookie with normalized values
- Manages `current_company_id` cookie for single company selection
- Clears inconsistent state automatically

#### 3. Company Selector (`components/settings/companies/CompanySelector.tsx`)
Main UI component providing:
- Dropdown interface for scope selection
- Search functionality for companies
- Visual indicators for current scope
- Responsive state management
- Automatic hiding for company users

#### 4. AppDataProvider Integration (`contexts/AppDataProvider.client.tsx` & `contexts/AppDataProvider.server.tsx`)
Provides centralized state management:
- Server-side scope detection from cookies
- Client-side reactive state updates
- Impersonation context handling
- Company user detection and restrictions

### Data Filtering Implementation

#### 1. Cookie-Based Scope Detection (`lib/supabase/queries.server.ts`)
```typescript
export async function getCompanyScopeFromCookies(
  cookieStore?: ReadonlyRequestCookies,
): Promise<CompanyScopeFilter>
```

This function:
- Reads scope and company ID from cookies
- Retrieves assigned company IDs for "assigned" scope
- Returns normalized scope filter object
- Handles user permissions automatically

#### 2. Query Filtering Patterns

**Server-Side Filtering (Preferred):**
```typescript
// Single Company Filtering
if (filter.scope === "single" && filter.companyId) {
  query = query.eq("company_id", filter.companyId);
}

// Assigned Companies Filtering
if (filter.scope === "assigned" && (filter.companyIds?.length ?? 0) > 0) {
  query = query.in("company_id", filter.companyIds);
}

// Global View - No additional filtering returns all tenant data
```

**Client-Side Filtering (Minimal Use):**
```typescript
// Only filter in single company mode - other modes already filtered server-side
const scopedData = companyScope === "single" && selectedCompany?.id
  ? data.filter((item) => item.company_id === selectedCompany.id)
  : data; // For "global" and "assigned", data is already filtered server-side
```

**Enhanced Meeting Queries (v1.1+):**
```typescript
fetchMeetingsServer(
  tenantId,
  scope.scope === "single" ? scope.companyId ?? undefined : undefined,
  scope.scope === "assigned" ? scope.companyIds : undefined,
  cookieStore
)
```

### Page Integration

All major application pages have been updated to support company scoping:

#### Dashboard & Overview Pages
- `app/(auth)/company_overview/page.tsx`
- `app/(auth)/company_overview/audit/page.tsx`

#### Feature-Specific Pages
- **Training**: `app/(auth)/training/page.tsx`, `/enrollments`, `/analytics`, `/reporting`
- **Planner**: `app/(auth)/planner/page.tsx`, `/list`, `/analytics`
- **Budgets**: `app/(auth)/budgets/page.tsx`, `/products`
- **Compliance**: `app/(auth)/compliance/page.tsx`
- **Calendar**: `app/(auth)/calendar/page.tsx`, `/list`
- **Reporting**: `app/(auth)/reporting/page.tsx`
- **M365**: `app/(auth)/m365/secure-score/page.tsx`

#### Integration Pattern
Each page follows this pattern:
1. Import `getCompanyScopeFromCookies` helper
2. Extract scope information from cookies
3. Pass scope filter to data fetching functions
4. Apply appropriate filtering based on scope

## User Experience

### Navigation Flow
1. User selects desired scope from company selector dropdown
2. System updates cookies via server action
3. Page refreshes to apply new scope
4. All subsequent data requests use the selected scope
5. Scope persists across page navigation

### Visual Indicators
- **Global View**: Globe icon + "Global View" text
- **Single Company**: Company name display
- **Assigned Companies**: "Assigned Companies (N)" with count
- **Current Selection**: Check mark indicator in dropdown

### Search Functionality
- Real-time search within company selector
- Filters available companies by name
- Maintains scope selection during search
- Clears search on selection

## Security Considerations

### Row Level Security (RLS)
- Database RLS policies automatically enforce tenant-level restrictions
- Company user restrictions enforced at application layer
- Impersonation context properly isolated

### Permission-Based Access
- AppDataProvider includes permission checking: `hasPermission(permission)`
- Company assignments verified through database queries
- Scope changes validated against user permissions

### Cookie Security
- Cookies set with `sameSite: "lax"` for CSRF protection
- Path restriction to "/" for application-wide scope
- Automatic cleanup of inconsistent state

## Database Dependencies

### Required Tables/Views
- `companies` - Company data with tenant_id filtering
- `users` - User profiles with company assignments
- `v_user_effective_permissions` - Permission checking
- Feature-specific tables with `company_id` foreign keys

### Query Optimization
- Indexes on `company_id` columns for efficient filtering
- Cached queries using `unstable_cache` with appropriate tags
- Batch queries where possible to reduce database load

## Migration Notes

### Removed Components
- `CompanyRoleManagerDialog.tsx` - Replaced with integrated company selector functionality

### Updated Dependencies
- All page components updated to use scope-aware data fetching
- Query functions enhanced with company filtering
- AppDataProvider expanded with company context

## Usage Examples

### For Developers

#### Basic Page Integration
```typescript
import { getCompanyScopeFromCookies } from "@/lib/supabase/queries.server";

export default async function MyPage() {
  const cookieStore = await cookies();
  const scope = await getCompanyScopeFromCookies(cookieStore);

  const data = await fetchDataWithScope(scope);

  return <MyComponent data={data} />;
}
```

#### Custom Query Filtering
```typescript
function applyCompanyFilter(query: QueryBuilder, scope: CompanyScopeFilter) {
  if (scope.scope === "single" && scope.companyId) {
    return query.eq("company_id", scope.companyId);
  } else if (scope.scope === "assigned" && scope.companyIds?.length) {
    return query.in("company_id", scope.companyIds);
  }
  return query; // Global view - no filtering
}
```

### For Users

#### Switching Company Scope
1. Click the company selector dropdown in the navigation
2. Choose from:
   - "Global View" for all companies
   - "Assigned Companies" for your assigned set
   - Individual company names for single company focus
3. Page automatically refreshes with new scope applied

#### Managing Company Assignments
- Company assignments are managed through user roles and permissions
- Contact administrators to modify company assignments
- Assignments automatically sync with "Assigned Companies" view

## Testing Considerations

### Functional Testing
- Test all three scoping modes across different pages
- Verify proper data isolation between companies
- Confirm company user restrictions work correctly
- Test impersonation scenarios

### Performance Testing
- Monitor query performance with company filtering
- Verify cache effectiveness across scope changes
- Test with large numbers of companies

### Security Testing
- Verify users cannot access unauthorized company data
- Test scope switching with different permission levels
- Confirm proper cookie handling and security

## Future Enhancements

### Potential Improvements
1. **Bulk Company Selection**: Allow multiple specific companies (beyond assigned)
2. **Scope Bookmarking**: Save frequently used scopes as bookmarks
3. **Company Hierarchies**: Support parent/child company relationships in scoping
4. **Advanced Filtering**: Add additional filtering criteria within scopes
5. **Scope Analytics**: Track usage patterns across different scoping modes

### API Extensions
1. **Public API Integration**: Extend company scoping to public API endpoints
2. **GraphQL Support**: Add company scoping to GraphQL queries
3. **Webhook Filtering**: Apply company scoping to webhook payloads

## Troubleshooting

### Common Issues

#### Scope Not Persisting
- Check cookie settings and browser storage
- Verify server action is completing successfully
- Ensure page refresh occurs after scope change

#### Data Not Filtering
- Confirm query functions are using scope parameter
- Check database column names match filtering logic
- Verify RLS policies are not interfering

#### Company Selector Not Showing
- Verify user has access to multiple companies
- Check if user is a company user (selector hidden for them)
- Confirm user permissions allow company switching

### Debug Information
- Use browser developer tools to inspect cookies
- Check server logs for scope-related errors
- Verify database queries include proper filtering

## Recent Updates

### Version 1.1 (2025-09-15): Consistency Fixes
- **Client-Side Filtering Logic**: Fixed inconsistent filtering logic in overview components
- **Calendar Integration**: Added assigned companies support to calendar pages
- **Server Query Enhancement**: Extended `fetchMeetingsServer` with multi-company filtering
- **Data Flow Optimization**: Improved server-side vs client-side filtering distribution
- **Performance Improvements**: Better cache utilization and reduced redundant filtering

See [Changelog: Company Scoping Consistency Fixes](CHANGELOG-2025-09-15-company-scoping-consistency-fixes.md) for detailed technical information.

## Related Documentation

- [Company Scoping Consistency Fixes Changelog](CHANGELOG-2025-09-15-company-scoping-consistency-fixes.md)
- [User Management and Permissions](user-management.md)
- [Database Architecture](database-architecture.md)
- [Authentication and Authorization](authentication.md)
- [Impersonation System](impersonation.md)