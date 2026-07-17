# Authentication & Error Handling Report

## Authentication Approach

The assignment specified using OAuth2 Authorization Code flow with the provided authorization and token endpoints. However, those endpoints were placeholders and could not be used to establish a real OAuth2 connection in n8n. Because of this limitation, I implemented the workflow using a Postman Mock Server to simulate the CRM API and authentication process.

Instead of OAuth2 credentials, I configured an **HTTP Header Authentication** credential in n8n. The credential automatically adds an `Authorization` header containing a mock bearer token to every API request. This allowed me to keep the workflow structure close to a real authenticated API integration while using fully functional mock endpoints for testing.

## Security Considerations

The authorization token is stored inside n8n's credential manager rather than hardcoded in workflow nodes. This keeps authentication details separate from the workflow and prevents sensitive values from being exposed when exporting or sharing the workflow. The workflow never returns authentication tokens in responses or logs.

## Error Handling Strategy

The workflow first searches for a contact by email. If a contact exists, it is updated; otherwise, a new contact is created. The Create Contact request is configured to continue on failure so that a **409 Conflict** (duplicate email) can be handled gracefully. When a duplicate is detected, the workflow searches for the contact again and updates the existing record instead of failing. Successful operations are logged as **created**, **updated**, or **updated_after_duplicate** and returned to the caller. Other API failures are detected through conditional checks and return an appropriate 500 error response, ensuring the workflow handles expected and unexpected errors reliably.