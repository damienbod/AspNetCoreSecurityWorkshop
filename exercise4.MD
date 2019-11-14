
# Add specific Authorization for an ASP.NET Core razor page

Test user : alice@alice.ch
Test password: Alice!01

Test user : bob@bob.ch
Test password: Bobby!01

## Exercise Goal

The aim of this exerise is to learn how to implement ASP.NET Core authorization using an AuthorizationHandler and a Policy.

## Description

The Razor Page BobsProtected should only be visible for people with bob in the email. This will be implemented using a IAuthorizationRequirement and a AuthorizationHandler

### Step 1 

Create a new class which implements the IAuthorizationRequirement in the RazorPageOidcClient project

```csharp
using Microsoft.AspNetCore.Authorization;
 
namespace RazorPageOidcClient
{
    public class IsAdminRequirement : IAuthorizationRequirement{}
}
```

### Step 2 

Create a handler for the requirement and add some validation logic.

```csharp
using Microsoft.AspNetCore.Authorization;
using System.Threading.Tasks;

namespace RazorPageOidcClient
{
    public class BobIsAnAdmin : AuthorizationHandler<IsAdminRequirement>
    {
        protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, 
            IsAdminRequirement requirement)
        {
            if (!string.IsNullOrEmpty(context.User.Identity.Name) && 
                context.User.Identity.Name.ToLower().Contains("bob"))
            {
                context.Succeed(requirement);
            }

            return Task.CompletedTask;
        }
    }
}

```

### Step 3

Register the Handler and use the requirement

```csharp
public void ConfigureServices(IServiceCollection services)
{
	// ... 

	services.AddSingleton<IAuthorizationHandler, BobIsAnAdmin>();

	services.AddAuthorization(options =>
	{
		options.AddPolicy("IsBobRequirementPolicy", policyIsAdminRequirement =>
		{
			policyIsAdminRequirement.Requirements.Add(new IsAdminRequirement());
		});
	});

	services.AddRazorPages();
}
```

### Step 4

Add the Authorize attribute with the Policy

```csharp
[Authorize("IsBobRequirementPolicy")]
public class BobsProtectedModel : PageModel
```

### Step 5

Hide menu for the bob admin page, if the user is not authorized in the _Layout.cshtml.

```csharp
@using Microsoft.AspNetCore.Authorization
@inject IAuthorizationService AuthorizationService
```
and 

```csharp
 @if ((await AuthorizationService.AuthorizeAsync(User, "IsBobRequirementPolicy")).Succeeded)
{
	<li class="nav-item">
		<a class="nav-link text-dark" asp-area="" asp-page="/BobsProtected">BobsProtected</a>
	</li>
}

```

### Authorisation using RequireClaim

You could also implement this use case using the RequireClaim instead of creating a specific handler.

```csharp
services.AddAuthorization(options =>
{
	options.AddPolicy("IsBobRequirementPolicy", policyIsAdminRequirement =>
	{
		policyIsAdminRequirement.RequireClaim("name", "bob@bob.ch");
	});
});
			
```
			

			
			
			
			
			