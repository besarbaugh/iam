# Sandbox validation checklist

Use only a disposable personal sandbox tenant and application. Never commit IDs, tokens, secrets,
certificates, exports, or screenshots.

## Static checks

- [ ] The JSON parses successfully.
- [ ] Every action in the design's forbidden list is absent.
- [ ] `applications/permissions/update` is present.
- [ ] `servicePrincipals/appRoleAssignedTo/update` is present.
- [ ] No action can create or assign Entra roles.
- [ ] The role assignment is scoped to one application, not `/`.

## Positive capability tests

- [ ] Read application and enterprise-application configuration.
- [ ] Update branding and basic properties.
- [ ] Update redirect URIs and authentication settings.
- [ ] Update the supported account audience.
- [ ] Add and remove a disposable credential.
- [ ] Update requested API permissions without granting admin consent.
- [ ] Create, modify, disable, and remove a disposable app-role definition.
- [ ] Assign and remove a test user or group from an enterprise-app role.

## Negative capability tests

The API must reject each operation; disabled portal controls alone are insufficient evidence.

- [ ] Add an owner to the application object.
- [ ] Add an owner to the service-principal object.
- [ ] Assign this role or another Entra role to a second principal.
- [ ] Modify the custom role definition.
- [ ] Create another app registration.
- [ ] Delete the application object.
- [ ] Delete the service-principal object.

## Migration gate

- [ ] The Operator can activate the role at the exact application scope.
- [ ] Effective-access review found no broader inherited administrative path.
- [ ] Positive and negative tests passed on a disposable application.
- [ ] Recovery through Privileged Role Administrator is documented.
- [ ] Only then remove the final user or service-principal owner.
- [ ] Verify the application and enterprise application have zero owners.
