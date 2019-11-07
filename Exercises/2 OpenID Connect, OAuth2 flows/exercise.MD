
# Create an Open ID Connect Code Flow client with Razor Pages

Test user : alice@alice.ch
Test password: Alice!01

Test user : bob@bob.ch
Test password: Bobby!01

## Exercise Goal

The aim of this exerise is to learn how to implement an OpenID Connect Code flow authentication in an ASP.NET Core Razor Page application.

## Step 1 

Add the required nuget packages to the RazorPageOidcClient project.

```xml
<PackageReference Include="Microsoft.AspNetCore.Authentication.OpenIdConnect" Version="3.0.0" />
```

## Step 2 

Configure the Startup authentication. The configuration must exactly match the configuration in the STS. 

```csharp
services.AddAuthentication(options =>
{
	options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
	options.DefaultChallengeScheme = OpenIdConnectDefaults.AuthenticationScheme;
})
.AddCookie()
.AddOpenIdConnect(options =>
{
	options.SignInScheme = "Cookies";
	options.Authority = "https://localhost:44318";
	options.RequireHttpsMetadata = true;
	options.ClientId = "codeflowpkceclient";
	options.ClientSecret = "codeflow_pkce_client_secret";
	options.ResponseType = "code";
	options.UsePkce = true;
	options.Scope.Add("profile");
	options.Scope.Add("offline_access");
	// add this if the user claims are not included in the id_token
	// options.GetClaimsFromUserInfoEndpoint = true;
	options.SaveTokens = true;
	options.TokenValidationParameters = new TokenValidationParameters
	{
		NameClaimType = "name"
	};
});
```

The UseRouting must be implemented before the UseAuthentication and the UseAuthorization

```
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{

	JwtSecurityTokenHandler.DefaultInboundClaimTypeMap.Clear();

	if (env.IsDevelopment())
	{
		app.UseDeveloperExceptionPage();
	}
	else
	{
		app.UseExceptionHandler("/Error");
		app.UseHsts();
	}

	app.UseHttpsRedirection();
	app.UseStaticFiles();

	app.UseRouting();

	app.UseAuthentication();
	app.UseAuthorization();

	app.UseEndpoints(endpoints =>
	{
		endpoints.MapRazorPages();
	});
}
```

## Step 3 

Add the Authorize attribute to the protected razor pages, ie  the Index.cshtml.cs file

```
[Authorize]
```

## Step 4 

Add a new Logout and SignedOut Razor page to the client project

Implement the sign out page

```csharp
[Authorize]
public class LogoutModel : PageModel
{
	public async Task<IActionResult> OnGetAsync()
	{
		await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
		return Redirect("/SignedOut");
	}
}
```

## Step 5 

Add the user email to the UI, display logout button only when logged in

```
@if (Context.User.Identity.IsAuthenticated)
{
	<li class="nav-item">
		<a class="nav-link text-dark" asp-area="" asp-page="/Logout">Logout</a>
	</li>

	<span class="nav-link text-dark">Hi @Context.User.Identity.Name</span>
}
else
{
	<li class="nav-item">
		<a class="nav-link text-dark" asp-area="" asp-page="/Index">Login</a>
	</li>
}
```

### Step 8

Run the applications in IIS Express. Start both projects in Visual Studio.


