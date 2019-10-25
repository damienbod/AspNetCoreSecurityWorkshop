Workshop Requirements

- PC with .NET Core 3 SDK and Visual Studio 2019/Visual Studio Code installed.
- NodeJS
- Internet WLAN connection
 
[Installation](https://github.com/damienbod/AspNetCoreSecurityWorkshop/blob/master/Exercises/installation.md) 

## 9:00 - 10:30

### Intro
	- Introduction round
	- Agenda
	- Security requirements

### ASP.NET Core Framework Security features
	- Claims, Principals, Identities, Claims based Identity
	- Cookie Authentication
	- Data Protection
	- Authorization 
	- External Authentication Providers
	- User Secrets
	- FIDO2
	
### Exercise:
	- ASP.NET Core Identity MFA
	
## 11:00 - 12:30
	
### OpenID Connect, OAuth2 flows
	- OpenID Connect, OAuth2
	- OAuth2 Resource Owner Credentials Flow
	- OpenID Connect Code flow + PKCE + secret
	- OpenID Connect Hybrid flow
	- OpenID Connect Authorization Code flow + PKCE - secret
	- OAuth Device Flow

### Exercise
	- IdentityServer4 secure token service with an ASP.NET Core OpenID Connect Code flow client + PKCE
	
## Lunch

## 14:00 - 15:30

### API Authorization
	- APIs with tokens authorization
	- APIs with cookies authorization
	- Introspection
	- Public, protected APIs

### Exercise
	- Client/API with JWT Bearer token authorization

### Authorization policies, claims
	- Policies
	- Handlers
	- Requirements
	- Custom authorization
	 
### Exercise
	- Implementing authorization using claims, policies, handlers
	
## 16:00 - 17:30

### Protecting the session, client
	- Click jacking
	- XSS
	- CSRF
	- CSP
	- HSTS
	- Cookie protection


