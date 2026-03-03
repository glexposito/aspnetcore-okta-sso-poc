# OktaSso.Poc

Minimal ASP.NET Core MVC sample with:

- Local cookie auth (`user` / `123`) for quick testing
- Okta OIDC SSO (`Login with Okta`)
- Anonymous page and protected pages

## Tech Stack

- .NET 10 (`net10.0`)
- ASP.NET Core MVC
- Cookie Authentication + OpenID Connect

## Project Structure

- `OktaSso.Poc.sln`
- `OktaSso.Poc.Web/`

Key files:

- `OktaSso.Poc.Web/Program.cs` - auth pipeline and Okta OIDC config
- `OktaSso.Poc.Web/Controllers/HomeController.cs` - local login, logout, Okta challenge
- `OktaSso.Poc.Web/Views/Home/Login.cshtml` - local login + Okta button
- `OktaSso.Poc.Web/Views/Home/Anonymous.cshtml` - public page

## Authentication Flow

- All `HomeController` actions are protected with `[Authorize]`.
- Public actions use `[AllowAnonymous]` (`Login`, `Anonymous`, `OktaLogin`, `Error`).
- Local login checks hardcoded credentials:
  - Username: `user`
  - Password: `123`
- Okta login uses OIDC challenge and callback at `/signin-oidc`.

## Run Locally

From repo root:

```bash
dotnet restore
dotnet build OktaSso.Poc.sln
dotnet run --project OktaSso.Poc.Web
```

Default URLs (from launch settings):

- `https://localhost:7238`
- `http://localhost:5224`

## Test Endpoints

- Login page: `https://localhost:7238/Home/Login`
- Public page: `https://localhost:7238/Home/Anonymous`
- Protected home: `https://localhost:7238/Home/Index`

## Okta Configuration

### 1. Create Okta OIDC Web App

In Okta Admin:

1. `Applications -> Applications -> Create App Integration`
2. Choose `OIDC - OpenID Connect`
3. Application type: `Web`
4. Sign-in redirect URI:
   - `https://localhost:7238/signin-oidc`
5. Sign-out redirect URI:
   - `https://localhost:7238/`
6. Save and copy:
   - Client ID
   - Client Secret

### 2. Add App Settings

Set in `OktaSso.Poc.Web/appsettings.json` (or better: user-secrets/environment variables):

```json
"Okta": {
  "Domain": "https://your-org.okta.com",
  "ClientId": "your-client-id",
  "ClientSecret": "your-client-secret"
}
```

Notes:

- Use your org domain, not `-admin` URL.
- This app uses issuer `https://<domain>/oauth2/default`.

### 3. Assign Users to App

- `Applications -> OktaSso.Poc.Web -> Assignments`
- Assign your test user (or group).

### 4. Configure Authorization Server Policy

For `Security -> API -> Authorization Servers -> default -> Access Policies`:

1. Add policy (target your app client)
2. Add rule with:
   - Grant type: `Authorization Code`
   - User condition: assigned users
   - Scopes: allow at least `openid`, `profile`, `email`

Without policy/rule, Okta returns `400 access_denied`.

## Expected SSO Behavior

`Login with Okta` always redirects to Okta first:

- If no Okta session: user enters credentials
- If Okta session exists: SSO returns immediately (no prompt)

## Common Issues

### `400 Bad Request` / `access_denied` in Okta

Usually one of:

- Missing access policy/rule on `default` authorization server
- User not assigned to app
- Wrong domain/issuer

### Callback mismatch

If Okta says redirect URI mismatch, ensure both sides use exactly:

- `https://localhost:7238/signin-oidc`

### Build error `NETSDK1226`

This project currently includes:

```xml
<AllowMissingPrunePackageData>true</AllowMissingPrunePackageData>
```

in `OktaSso.Poc.Web.csproj` as a local SDK workaround.

## Security Notes

- Local hardcoded credentials are for demo only.
- Replace with DB + hashed passwords for real usage.
- Do not commit real client secrets to source control.
  - Prefer environment variables or `dotnet user-secrets`.

## Suggested Next Steps

1. Move Okta secrets out of `appsettings.json`.
2. Replace hardcoded local auth with DB-backed users + password hashing.
3. Add integration tests for login/redirect flows.
