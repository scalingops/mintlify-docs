# Changelog - September 14, 2025: Device Mapping System & Toast Notification Fixes

## Summary

This release introduces a comprehensive device type mapping system for Meraki integrations, fixes toast notification issues across the application, and includes critical database security improvements.

## 🚀 New Features

### Meraki Device Type Mapping System

**Added complete device type mapping functionality for Meraki integrations:**

- **New Page**: `app/(auth)/settings/integrations/meraki/device-mappings/page.tsx`
  - Server-rendered page for managing Meraki device type mappings
  - Fetches tenant-level Meraki integration data
  - Retrieves device types and existing mappings
  - Supports both discovered and known Meraki product types

- **New Component**: `components/settings/integrations/DeviceTypeMappingPage.client.tsx`
  - Client-side interactive mapping interface
  - Real-time mapping updates with optimistic UI
  - Bulk save functionality for all mappings
  - Refresh capability to discover new external device types
  - Statistics overview showing mapping status

- **New Component**: `components/settings/device-types/DeviceTypeSelect.tsx`
  - Reusable device type selection component
  - Supports creating new device types inline
  - Includes "No mapping" option for unmapped types
  - Integration with existing AddDeviceTypeDialog

- **New Component**: `components/ui/HeaderSinglePage.tsx`
  - Standardized header component for single-page workflows
  - Supports back navigation and action buttons
  - Used across mapping and other single-page interfaces

**Key Features:**
- Maps external device types (Meraki product types) to internal device types
- Auto-discovery of device types from synced devices
- Support for known Meraki types even without devices
- Real-time mapping statistics and validation
- Optimistic UI updates with proper error handling

## 🐛 Bug Fixes

### Toast Notification API Fixes

**Fixed toast notification usage across multiple components:**

**Fixed Components:**
- `app/(auth)/settings/integrations/syncro/[id]/devices/SyncroDeviceImportClient.tsx`
- `components/goals/GoalsPage.client.tsx`
- `components/settings/integrations/MerakiSettings.client.tsx`
- `components/tickets/new/NewTicketAIPage.client.tsx`
- `components/tickets/template/TemplateEditPage.client.tsx`

**Changes Made:**
- Updated from old shadcn/ui toast pattern: `toast({ title: "Success", description: "Message" })`
- To correct toast API: `toast.success("Message")`, `toast.error("Message")`, `toast.info("Message")`
- Ensured proper import: `const toast = useToast()` (not destructured)
- Consistent error handling and user feedback

**Benefits:**
- Eliminates broken toast notifications
- Provides consistent user feedback across the application
- Follows the established toast API patterns from CLAUDE.md guidelines

## 🔧 Enhancements

### Enhanced Integration Device Mappings

**Updated `app/actions/integration-device-mappings.ts`:**
- Enhanced server actions for device type mapping management
- Better error handling and validation
- Support for multiple integration types (Syncro, Meraki, etc.)
- Improved data fetching with proper tenant scoping

### Meraki Sync Function Improvements

**Updated `supabase/functions/sync-meraki-devices/index.ts`:**
- Added device type mapping lookup during sync
- Enhanced device categorization based on mappings
- Better error handling for missing mappings
- Support for tenant-specific device type assignments

## 🔒 Security & Database Improvements

### Database Function Security Hardening

**New Migration**: `supabase/migrations/20250914_fix_function_search_path_and_move_vector.sql`

**Key Security Enhancements:**
- Hardened search_path for all database functions and procedures
- Moved vector extension to dedicated `extensions` schema
- Set explicit search_path for all user-defined functions
- Added security validation for SECURITY DEFINER functions
- Reduced schema hijack risks

**Changes Made:**
- Created and secured `extensions` schema
- Moved `vector` extension from `public` to `extensions` schema
- Set search_path to `pg_catalog, pg_temp, public, util, extensions` for all functions
- Added role-level search_path defaults for `anon`, `authenticated`, `service_role`
- Validation queries to ensure security compliance

### RLS Policy Migration Enhancement

**Updated Migration**: `supabase/migrations/20250914_migrate_rls_to_can.sql`

**Improvements:**
- Enhanced RLS policy migration to standardized `can()`/`can_with_company()` pattern
- Better snapshot handling for existing policies
- Improved schema detection and column validation
- More robust error handling and rollback capabilities

## 📋 Technical Details

### Database Schema Changes

**New Table Structure for Device Mappings:**
- Utilizes existing `integration_device_type_mappings` table
- Links `external_type` (Meraki product type) to `device_type_id`
- Tenant-scoped with proper RLS policies
- Supports multiple integration providers

### API Enhancements

**New Server Actions:**
- `updateIntegrationDeviceTypeMappingAction()` - Create/update device type mappings
- `getExternalDeviceTypesAction()` - Fetch external device types for refresh functionality

### Component Architecture

**Reusable Components:**
- `DeviceTypeSelect` - Standardized device type selection with inline creation
- `HeaderSinglePage` - Consistent single-page layout header
- `DeviceTypeMappingPage` - Generic mapping interface (can be extended for other integrations)

## 🧪 Testing Considerations

**Areas to Test:**
1. **Meraki Device Mapping:**
   - Navigate to Meraki integration settings
   - Access device mappings page
   - Create and update device type mappings
   - Refresh external device types
   - Verify mapping persistence

2. **Toast Notifications:**
   - Test all fixed components for proper toast display
   - Verify success/error messages appear correctly
   - Check toast styling and positioning

3. **Database Functions:**
   - Verify all database functions execute without search_path issues
   - Test vector extension functionality
   - Validate RLS policy enforcement

## 🔄 Migration Notes

**For Existing Deployments:**
1. Run migrations in order: `20250914_fix_function_search_path_and_move_vector.sql` first
2. Then run: `20250914_migrate_rls_to_can.sql`
3. Verify no existing toast notifications are broken
4. Test Meraki device mapping functionality if using Meraki integration

**Breaking Changes:**
- Vector extension moved to `extensions` schema (should be transparent)
- Toast API usage updated (existing code using old pattern will break)

## 📝 Documentation Updates

**Files Added:**
- Device mapping workflow documentation (implicit in component structure)
- Enhanced error handling patterns for toast notifications

**Files Modified:**
- CLAUDE.md guidelines remain consistent with toast API usage requirements

---

*This changelog documents significant improvements to the device management system, user experience through proper notifications, and critical security enhancements to the database layer.*