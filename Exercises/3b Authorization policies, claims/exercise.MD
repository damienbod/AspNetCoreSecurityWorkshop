
# Add specific Authorization for an ASP.NET Core razor page

Test user : alice@alice.ch
Test password: Alice!01

Test user : bob@bob.ch
Test password: Bobby!01

## Description

The Razor Page BobsProtected should only be visible for people with bob in the email. WThis will be implemented using a IAuthorizationRequirement and a AuthorizationHandler

### Step 1 

Create a new class which implements the IAuthorizationRequirement

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

Hide menu for the bob admin page, if the user is not authorized.

```csharp

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
			

			
			
			
			
			