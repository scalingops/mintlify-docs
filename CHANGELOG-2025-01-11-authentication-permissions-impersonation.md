# Changelog - January 11, 2025

## Authentication, Permissions, and Impersonation System Enhancements

### 🚀 Major Features

#### Enhanced Impersonation System
- **Permission Context Switching**: Impersonation now uses target user's permissions for route access control
- **Company Data Isolation**: When impersonating company users, system locks to target company's data
- **Read-Only Enforcement**: Most operations are restricted during impersonation to prevent data corruption
- **Cookie Synchronization**: Company selection automatically syncs to impersonated company context
- **Route Restrictions**: Sensitive admin routes (training/compliance settings) blocked during impersonation
- **Notes Sync Prevention**: Ticket notes synchronization disabled during impersonation for security

#### Permission System Standardization
- **Training Permission Update**: Changed from `manage_training` to `manage_settings_training` for consistency
- **Route-Based Access Control**: Enhanced middleware checks permissions using target user context during impersonation
- **Company User Restrictions**: Company users blocked from accessing training and compliance settings regardless of permissions

#### Company Data Scoping Improvements
- **Defense-in-Depth Security**: Multiple layers ensure company users only access their company data
- **Automatic Company Detection**: Fallback logic detects company context from user profile when cookies missing
- **Server-Side Enforcement**: API endpoints now enforce company scoping at the database level
- **Profile-Based Scoping**: User profiles determine data access scope rather than client-provided parameters

### 🔧 Technical Improvements

#### Middleware Enhancements
- **Impersonation-Aware Routing**: Route permissions evaluated using impersonated user's context
- **Dynamic User Resolution**: Middleware reads impersonation context to determine effective user
- **Enhanced Security Checks**: Company user type validation prevents unauthorized access to sensitive areas
- **CORS and JSON Response Handling**: Improved detection of API vs navigation requests

#### Authentication Flow Updates
- **Server Action Integration**: Better integration with server actions for mutation operations
- **Session Context Management**: Enhanced user context resolution for both regular and impersonated sessions
- **Multi-User Profile Support**: System handles both target and source user profiles during impersonation

#### UI Component Improvements
- **Locked Company Selector**: New component for company users showing read-only company selection
- **Conditional Company Selection**: TopBar dynamically shows appropriate company selector based on user type
- **Impersonation Status Indicators**: Components aware of impersonation state for conditional behavior

### 📊 Data Access Security

#### API Route Hardening
- **Profile-Based Authorization**: API routes use server-side user profile data instead of client parameters
- **Company Resolution**: Company names resolved server-side from database rather than trusting client input
- **Tenant Scoping**: All queries properly scoped to authenticated user's tenant

#### Database Query Optimization
- **Efficient Permission Loading**: User permissions and profile data fetched together
- **Company Filtering**: Database-level filtering ensures only authorized company data is returned
- **Role-Based Access**: Enhanced role resolution for both regular and impersonated contexts

### 🛡️ Security Enhancements

#### Impersonation Security Model
- **Allowlist-Based Operations**: Only specific, safe operations allowed during impersonation
- **Mutation Restrictions**: Write operations carefully controlled during impersonation sessions
- **Context Validation**: Impersonation context validated on every request
- **Session Isolation**: Clear separation between impersonator and target user data access

#### Permission Enforcement
- **Middleware-Level Checks**: Permissions validated before reaching application logic
- **Multi-Layer Validation**: Both route-level and component-level permission checks
- **Dynamic Permission Context**: Permissions adapt based on impersonation status

### 📦 Modified Files

#### Core Authentication & Middleware
- `middleware.ts` - Enhanced impersonation handling and JSON response detection
- `lib/supabase/middleware.ts` - Impersonation-aware permission checking and company user restrictions
- `lib/auth/route-permissions.ts` - Updated training permission requirement

#### Server Context & Providers  
- `contexts/AppDataProvider.server.tsx` - Enhanced company scoping and user profile resolution
- `components/shared/auth-layout/TopBar/TopBar.tsx` - Conditional company selector rendering

#### Page Components
- `app/(auth)/budgets/page.tsx` - Company context fallback logic for company users
- `app/(auth)/compliance/page.tsx` - Enhanced company scoping and data filtering
- `components/settings/settings-sidebar.tsx` - Updated permission requirement for training settings

#### API & Data Access
- `app/api/tickets/route.ts` - Server-side company scoping and profile-based authorization
- `components/tickets/TicketDetailsModal.tsx` - Impersonation-aware notes synchronization

### 🔄 Migration Notes

#### Permission Updates Required
- Any code checking for `manage_training` permission should be updated to `manage_settings_training`
- Review custom permission checks to ensure consistency with new naming convention

#### Impersonation Session Handling
- Components should use `useAppData()` hook to check `isImpersonating` status
- Server actions should validate impersonation context for write operations
- API routes should use server-side profile data rather than client-provided parameters

#### Company User Experience
- Company users will see locked company selection in UI
- Access to training and compliance settings automatically restricted
- Data automatically scoped to user's assigned company

### 🚨 Breaking Changes

#### Permission Name Change
- `manage_training` permission renamed to `manage_settings_training`
- Existing code referencing the old permission name will need updates

#### API Parameter Changes
- Tickets API no longer accepts client-provided `tenantId` parameter
- Company scoping now determined server-side from user profile
- Client-side company filtering parameters may be ignored for security

#### Impersonation Behavior
- Many operations become read-only during impersonation
- Notes synchronization disabled during impersonation
- Certain admin routes completely blocked during impersonation

---

*This update significantly enhances the security model for user impersonation, company data isolation, and permission management while maintaining a smooth user experience for both regular and impersonated sessions.*