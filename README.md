# IAM practice

Public, vendor-documented practice artifacts for cloud identity and access management.

## Entra app registration without standing owners

[`roles/contoso-app-reg-operator.json`](roles/contoso-app-reg-operator.json) defines a broad,
application-scoped Microsoft Entra custom role that can manage an app registration and assign
users or groups to app roles, but cannot:

- add or remove application or service-principal owners;
- create or delete applications;
- update catch-all privileged property sets; or
- create roles or delegate administrative access.

Read the [design notes](docs/app-reg-owner-replacement.md) and run the
[sandbox validation checklist](docs/validation-checklist.md) before adapting it.

This repository contains generic learning material only. It contains no tenant identifiers,
application identifiers, credentials, or organization-specific procedures.
