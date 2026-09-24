# Cloud Application Operator

Status: current design after PO review, 2026-09-24.

Replicate the Cloud Application Administrator surface, then drop the ability to grant
ownership and the ability to delete the application. Do not try to trim the Owner
property. Owner is an object ACE, not a role definition. Owners always get add-owners
and delete on that one app. Groups cannot be owners. Owner cannot be PIM'd.

Microsoft's documented stand-in for a "limited owner" is a custom role assigned at app
or directory scope. Cloud Application Administrator is the right template because it is
the full app-management permission set.

## Why Owner cannot be trimmed

| | Owner | Custom CAA minus owner/delete |
|---|---|---|
| Can trim permissions | No | Yes |
| Add other owners | Always | Excluded |
| Delete app / SP | Always | Excluded |
| PIM / time-bound | No | Yes |
| Assign to a group | No | Yes |
| Scope | That one app only | Tenant `/` or one app object ID |
| Sticky ACE on the object | Yes | No — role assignment only |
| Create new apps | No | Yes, if assigned at `/` with `applications/create` |

Owner exists on two objects. The application registration and the enterprise app
(service principal) have independent owners and independent delete. Exclude
`owners/update` and `delete` on both or the constraint is bypassed.

## Exclude

```text
microsoft.directory/applications/owners/update
microsoft.directory/applications.myOrganization/owners/update
microsoft.directory/servicePrincipals/owners/update
microsoft.directory/applicationPolicies/owners/update
microsoft.directory/applications/delete
microsoft.directory/applications.myOrganization/delete
microsoft.directory/servicePrincipals/delete
microsoft.directory/deletedItems.applications/delete
microsoft.directory/applications/createAsOwner
microsoft.directory/servicePrincipals/createAsOwner
microsoft.directory/applications/allProperties/update
microsoft.directory/applications.myOrganization/allProperties/update
microsoft.directory/servicePrincipals/allProperties/update
microsoft.directory/servicePrincipals/allProperties/allTasks
```

`allProperties/update` and `allProperties/allTasks` silently restore owners and delete.
Custom roles have no subtractive `excludedResourceActions`. Build a positive list.

Use `applications/create` rather than `createAsOwner`. That matches Cloud Application
Administrator: the assignee is not added as owner of apps they create.

## Include (practical custom-role set)

App registration:

```text
microsoft.directory/applications/create
microsoft.directory/applications/standard/read
microsoft.directory/applications/owners/read
microsoft.directory/applications/basic/update
microsoft.directory/applications/audience/update
microsoft.directory/applications/authentication/update
microsoft.directory/applications/credentials/update
microsoft.directory/applications/permissions/update
microsoft.directory/applications/disablement/update
microsoft.directory/applications/appRoles/update
microsoft.directory/applications/notes/update
microsoft.directory/applications/tag/update
microsoft.directory/applications/policies/update
microsoft.directory/applications/extensionProperties/update
microsoft.directory/applications/verification/update
microsoft.directory/applicationTemplates/instantiate
```

Enterprise app / service principal:

```text
microsoft.directory/servicePrincipals/create
microsoft.directory/servicePrincipals/standard/read
microsoft.directory/servicePrincipals/owners/read
microsoft.directory/servicePrincipals/basic/update
microsoft.directory/servicePrincipals/audience/update
microsoft.directory/servicePrincipals/authentication/update
microsoft.directory/servicePrincipals/credentials/update
microsoft.directory/servicePrincipals/permissions/update
microsoft.directory/servicePrincipals/policies/update
microsoft.directory/servicePrincipals/notes/update
microsoft.directory/servicePrincipals/tag/update
microsoft.directory/servicePrincipals/enable
microsoft.directory/servicePrincipals/disable
microsoft.directory/servicePrincipals/appRoleAssignedTo/read
microsoft.directory/servicePrincipals/appRoleAssignedTo/update
microsoft.directory/servicePrincipals/synchronization/standard/read
microsoft.directory/servicePrincipals/synchronizationJobs/manage
microsoft.directory/servicePrincipals/synchronizationSchema/manage
microsoft.directory/servicePrincipals/synchronizationCredentials/manage
microsoft.directory/applications/synchronization/standard/read
```

Consent (Graph only; the portal picker does not expose this action):

```text
microsoft.directory/servicePrincipals/managePermissionGrantsForAll.microsoft-application-admin
```

If Graph rejects an action that is not in the custom-role allowlist, drop that line.
The portal picker is the source of truth for what a custom role can hold.
Support tickets, Service Health, and M365 portal actions on Cloud Application
Administrator are not available to custom roles and will not come along.

Optional: `microsoft.directory/deletedItems.applications/restore` so they can recover
deletions they cannot perform.

Delete substitute is Disable (`applications/disablement/update` plus
`servicePrincipals/disable`).

## Caveats

- Still privileged. `credentials/update` lets the assignee add a secret and impersonate
  the app. "No owner / no delete" is not "cannot take over the app."
- `applications/create` only works when the assignment scope is `/`. Resource-scoped
  assignment cannot register new apps.
- If "Users can register applications" is On, users still become owners of apps they
  create. Turn that Off and use this role instead.
- Keep a break-glass Owner (or Privileged Role Administrator recovery path). Do not put
  the working team on the Owner ACE.
- PIM the assignment. Do not make it standing.
- Entra authorization is additive. This role cannot neutralize Cloud Application
  Administrator, Application Administrator, or an existing Owner ACE on the same person.

## Create the role

Portal: Entra ID → Roles and admins → New custom role. Search and check each include
action. You cannot clone a built-in role. Requires Entra P1/P2 and Privileged Role
Administrator.

Graph (includes the consent action the portal will not show):

```powershell
Connect-MgGraph -Scopes "RoleManagement.ReadWrite.Directory"

$rolePermissions = @{
  allowedResourceActions = @(
    "microsoft.directory/applications/create",
    "microsoft.directory/applications/standard/read",
    "microsoft.directory/applications/owners/read",
    "microsoft.directory/applications/basic/update",
    "microsoft.directory/applications/audience/update",
    "microsoft.directory/applications/authentication/update",
    "microsoft.directory/applications/credentials/update",
    "microsoft.directory/applications/permissions/update",
    "microsoft.directory/applications/disablement/update",
    "microsoft.directory/applications/appRoles/update",
    "microsoft.directory/applications/notes/update",
    "microsoft.directory/applications/tag/update",
    "microsoft.directory/applications/policies/update",
    "microsoft.directory/applications/extensionProperties/update",
    "microsoft.directory/applications/verification/update",
    "microsoft.directory/applicationTemplates/instantiate",
    "microsoft.directory/servicePrincipals/create",
    "microsoft.directory/servicePrincipals/standard/read",
    "microsoft.directory/servicePrincipals/owners/read",
    "microsoft.directory/servicePrincipals/basic/update",
    "microsoft.directory/servicePrincipals/audience/update",
    "microsoft.directory/servicePrincipals/authentication/update",
    "microsoft.directory/servicePrincipals/credentials/update",
    "microsoft.directory/servicePrincipals/permissions/update",
    "microsoft.directory/servicePrincipals/policies/update",
    "microsoft.directory/servicePrincipals/notes/update",
    "microsoft.directory/servicePrincipals/tag/update",
    "microsoft.directory/servicePrincipals/enable",
    "microsoft.directory/servicePrincipals/disable",
    "microsoft.directory/servicePrincipals/appRoleAssignedTo/read",
    "microsoft.directory/servicePrincipals/appRoleAssignedTo/update",
    "microsoft.directory/servicePrincipals/synchronization/standard/read",
    "microsoft.directory/servicePrincipals/synchronizationJobs/manage",
    "microsoft.directory/servicePrincipals/synchronizationSchema/manage",
    "microsoft.directory/servicePrincipals/synchronizationCredentials/manage",
    "microsoft.directory/servicePrincipals/managePermissionGrantsForAll.microsoft-application-admin"
  )
}

New-MgRoleManagementDirectoryRoleDefinition `
  -DisplayName "contoso-Cloud-Application-Operator" `
  -Description "Cloud App Admin minus owner assignment and delete" `
  -TemplateId (New-Guid).Guid `
  -IsEnabled `
  -RolePermissions $rolePermissions
```

JSON twin: [`roles/contoso-cloud-application-operator.json`](../roles/contoso-cloud-application-operator.json).

The narrower application-scoped role in [`roles/contoso-app-reg-operator.json`](../roles/contoso-app-reg-operator.json)
remains valid for a single-app operator who should not create apps. That is not the PO design.

## Test

Add owner and Delete must 403 on both the application and the service principal.
Disable, rotate a disposable secret, and edit redirect URIs must succeed.

See [validation-checklist.md](validation-checklist.md).

## Public Microsoft documentation

- [Cloud Application Administrator permissions](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/permissions-reference)
- [Custom permissions for app registrations](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-available-permissions)
- [Enterprise-app permissions for custom roles](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-enterprise-app-permissions)
- [App consent permissions for custom roles](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-consent-permissions)
- [Delegate app-registration permissions](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/delegate-app-roles)
- [Create a custom Entra role](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-create)
