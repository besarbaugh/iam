# Sandbox validation checklist

Use only a disposable personal sandbox tenant and application. Never commit IDs, tokens, secrets,
certificates, exports, or screenshots.

Two role variants live in this repo:

- `contoso-Cloud-Application-Operator` — PO design. CAA minus add-owner and delete. May include
  `applications/create`. Directory-scope `/` is expected if Operators should register apps.
- `contoso-AppReg-Operator` — narrower. One existing app only. Assignment scoped to one
  application, not `/`. Must not create apps.

## Static checks

- [ ] The JSON parses successfully.
- [ ] Every action in the design's forbidden list is absent.
- [ ] `applications/owners/update` and `servicePrincipals/owners/update` are absent.
- [ ] `applications/delete` and `servicePrincipals/delete` are absent.
- [ ] `allProperties/update` and `allProperties/allTasks` are absent.
- [ ] `createAsOwner` is absent on applications and service principals.
- [ ] `applications/permissions/update` is present.
- [ ] `servicePrincipals/appRoleAssignedTo/update` is present.
- [ ] No action can create or assign Entra roles.
- [ ] Scope matches the variant: `/` for Cloud Application Operator create; one application
      object ID for the narrower operator.

## Positive capability tests

- [ ] Read application and enterprise-application configuration.
- [ ] Update branding and basic properties.
- [ ] Update redirect URIs and authentication settings.
- [ ] Update the supported account audience.
- [ ] Add and remove a disposable credential.
- [ ] Update requested API permissions without granting admin consent.
- [ ] Create, modify, disable, and remove a disposable app-role definition.
- [ ] Assign and remove a test user or group from an enterprise-app role.
- [ ] Disable the application / service principal (delete substitute).
- [ ] Cloud Application Operator at `/` only: create a disposable app without becoming owner.

## Negative capability tests

The API must reject each operation; disabled portal controls alone are insufficient evidence.

- [ ] Add an owner to the application object.
- [ ] Add an owner to the service-principal object.
- [ ] Assign this role or another Entra role to a second principal.
- [ ] Modify the custom role definition.
- [ ] Delete the application object.
- [ ] Delete the service-principal object.
- [ ] Narrower operator only: create another app registration.

## Migration gate

- [ ] The Operator can activate the role at the intended scope.
- [ ] Effective-access review found no broader inherited administrative path.
- [ ] Positive and negative tests passed on a disposable application.
- [ ] Recovery through Privileged Role Administrator is documented.
- [ ] Only then remove the final user or service-principal owner.
- [ ] Verify the application and enterprise application have zero owners.
