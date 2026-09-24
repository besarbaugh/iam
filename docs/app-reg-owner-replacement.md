# Replace app-registration owners with scoped administration

## Status

2026-09-24: PO chose the Cloud Application Administrator template minus add-owner and
delete, not a trimmed Owner ACE and not this narrower application-scoped operator as the
primary design. Current write-up:
[cloud-application-operator.md](cloud-application-operator.md).

This page is the earlier application-scoped operator. It is still valid when the assignee
should manage one existing app and must not create apps directory-wide.

## Security invariant

An Operator may broadly administer one existing app registration and assign users or groups to
its app roles. The Operator cannot expand who administratively controls the application.

The role must not contain:

```text
microsoft.directory/applications/create
microsoft.directory/applications/createAsOwner
microsoft.directory/applications/delete
microsoft.directory/applications/owners/update
microsoft.directory/applications.myOrganization/owners/update
microsoft.directory/applications/allProperties/update
microsoft.directory/applications.myOrganization/allProperties/update
microsoft.directory/servicePrincipals/createAsOwner
microsoft.directory/servicePrincipals/delete
microsoft.directory/servicePrincipals/owners/update
microsoft.directory/servicePrincipals/allProperties/update
microsoft.directory/servicePrincipals/permissions/update
```

It must not contain role-definition, role-assignment, or PIM-administration actions.

Microsoft Entra custom roles do not support subtractive `excludedResourceActions`. The safe
construction is therefore a positive list of permitted property sets, not
`allProperties/update` followed by attempted exclusions.

## Important distinctions

- `applications/permissions/update` changes requested and exposed API permissions. Microsoft
  documents that it does not grant admin consent. Validate app-role-definition changes in a
  disposable sandbox application.
- `servicePrincipals/appRoleAssignedTo/update` assigns users and groups to app roles on the
  enterprise application. This grants runtime application access, not administrative control of
  the app registration.
- `applications/credentials/update` is deliberately powerful. Someone who can add a secret or
  certificate may impersonate the application. A stronger production design can separate
  credential rotation into another PIM-eligible role.
- `applications/create` is intentionally absent here. It is ineffective when the role is assigned at
  application scope and becomes broad creation authority if assigned at directory scope.
  The PO design in [cloud-application-operator.md](cloud-application-operator.md) includes
  `applications/create` and expects directory-scope assignment for a platform team.

## Assignment and migration

1. Create the custom role through an appropriately privileged administrative process.
2. Assign it through PIM at the scope of one disposable application.
3. Complete all positive and negative capability tests.
4. Assign the tested role at each application scope before removing the final owner.
5. Remove user owners.
6. Replace service-principal ownership with explicit OAuth permissions, then remove service
   principal owners.
7. Verify that both the application registration and local enterprise application are ownerless.

Entra authorization is additive. This role cannot neutralize broader permissions received through
another directory role, group membership, PIM assignment, or existing ownership. Review every
Operator's effective access before treating the control as complete.

## Public Microsoft documentation

- [Custom permissions for app registrations](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-available-permissions)
- [Delegate app-registration permissions](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/delegate-app-roles)
- [Enterprise-app assignment permissions](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-enterprise-apps)
- [Create a custom Entra role](https://learn.microsoft.com/en-us/entra/identity/role-based-access-control/custom-create)
- [Microsoft Entra PIM role assignments](https://learn.microsoft.com/en-us/entra/id-governance/privileged-identity-management/pim-how-to-add-role-to-user)
- [Unified role permission model](https://learn.microsoft.com/en-us/graph/api/resources/unifiedrolepermission)
