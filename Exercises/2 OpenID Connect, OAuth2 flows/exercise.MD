
# Create an Open ID Connect Code Flow client with Razor Pages

Test user : alice@alice.ch
Test password: Alice!01

Test user : bob@bob.ch
Test password: Bobby!01

## Step 1 

Add the required nuget packages

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

Add the Authorize attribute to the protected razor pacges

```
[Authorize]
```

## Step 4 

Add a logout razor page, signed out razor page

Add a logout and a signed out Razor page

Implement the sign out

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

