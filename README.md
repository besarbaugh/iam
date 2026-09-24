# IAM practice

Public, vendor-documented practice artifacts for cloud identity and access management.

## Entra app registration without standing owners

Current design after PO review (2026-09-24): replicate Cloud Application Administrator,
then remove add-owner and delete. Owner cannot be trimmed. See
[`docs/cloud-application-operator.md`](docs/cloud-application-operator.md) and
[`roles/contoso-cloud-application-operator.json`](roles/contoso-cloud-application-operator.json).

A narrower application-scoped operator remains in
[`roles/contoso-app-reg-operator.json`](roles/contoso-app-reg-operator.json). That role can
manage one existing app and assign users or groups to app roles, but cannot create apps,
add or remove owners, or delete the app. Design notes:
[`docs/app-reg-owner-replacement.md`](docs/app-reg-owner-replacement.md).

Run the [sandbox validation checklist](docs/validation-checklist.md) before adapting either
role.

This repository contains generic learning material only. It contains no tenant identifiers,
application identifiers, credentials, or organization-specific procedures.
