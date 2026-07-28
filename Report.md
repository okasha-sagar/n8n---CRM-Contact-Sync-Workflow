# OAuth2 Authentication & Error Handling Report

## OAuth2 Client Credentials Flow

The workflow authenticates using the OAuth2 Client Credentials grant. This is a two-party flow with no end-user login step: n8n sends a direct POST to the Token URL (`/oauth/token`) with the client ID and client secret; the authorization server validates them and returns an access token directly, with no authorization code or redirect involved. I used Client Credentials instead of the originally specified Authorization Code grant because the mock server can only return static, pre-defined responses — it cannot run a real interactive login page or issue a dynamic authorization code tied to a redirect. Client Credentials only requires the single token-endpoint call, which a static mock can represent faithfully; Authorization Code's browser-redirect step cannot be.

## n8n Configuration

I created an OAuth2 API credential with Grant Type: Client Credentials, the Access Token URL pointed at the mock server, Client ID/Secret entered directly into the credential (never in node parameters), and Scope `read:contacts write:contacts`. After connecting, n8n stored the returned access token encrypted in its credential store. Every HTTP Request node references this one credential via Generic Credential Type → OAuth2 API, so refresh is handled automatically before each call.

## Security Considerations

The client secret lives only inside the encrypted credential, never in an expression, parameter, or log line. Tokens are never logged, echoed in Set nodes, or returned in the webhook response — only actual contact data is. n8n re-requests a fresh access token automatically once the current one expires, so no plaintext token is ever re-entered manually.

## Error Handling Strategy

Only Create Contact runs with Continue On Fail enabled (full response, `neverError: true`), since it needs to inspect the response status explicitly rather than throw: a 409 is an expected race-condition outcome, not a failure, and routes to a re-search-and-update path; any other non-201 status returns a 400 with the error details directly to the caller. Every other HTTP node (Search, Update, Search Again, Update After Duplicate) uses default error behavior, so a real failure (400/401/500) halts that branch and is caught by a dedicated Error workflow, configured as this workflow's Error Workflow in its settings, rather than an inline Error Trigger node on the same canvas.

## Note on Endpoint Substitution

The assignment's specified CRM domain (`crm-mock-4271.api.example.io`) did not resolve when I attempted to connect, so I substituted a Postman Mock Server (same base path structure) to test end-to-end. Since Postman mocks can't validate real credentials, the `/oauth/token` endpoint returns a static canned token regardless of the client ID/secret sent — the credential and workflow configuration still follow the real OAuth2 flow described above, just against a server that can't perform genuine validation. I've flagged the unreachable domain to my mentor and am happy to re-point the credential at the original CRM URL if it's restored.
