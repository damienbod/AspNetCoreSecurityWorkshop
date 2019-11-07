
# Protect the API in the ASP.NET Core API project

Test user : alice@alice.ch
Test password: Alice!01

Test user : bob@bob.ch
Test password: Bobby!01

## Exercise Goal

The aim of this exerise is to learn how to secure an ASP.NET Core API which is used from a second Razor page application.

## Step 1 

Add the required nuget package to the WebApi project.

```xml
<PackageReference Include="IdentityServer4.AccessTokenValidation" Version="3.0.1" />
```

## Step 2 

Configure the WebApi project Startup authentication. The configuration must exactly match the configuration in the STS. 

```csharp
public void ConfigureServices(IServiceCollection services)
{
	services.AddAuthentication(IdentityServerAuthenticationDefaults.AuthenticationScheme)
	  .AddIdentityServerAuthentication(options =>
	  {
		  options.Authority = "https://localhost:44318";
		  options.ApiName = "ProtectedApiResource";
		  options.ApiSecret = "api_resource_in_protected_zone_secret";
		  options.RequireHttpsMetadata = true;
	  });

	services.AddAuthorization(options =>
		options.AddPolicy("protectedScope", policy =>
		{
			policy.RequireClaim("scope", "scope_used_for_api_in_protected_zone");
		})
	);

	services.AddControllers();
}
```

## Step 3

Remove the default Microsoft mappings in the WebApi project and add the UseAuthentication after the UseRouting()

```csharp
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

	app.UseStaticFiles();
	app.UseRouting();
	app.UseAuthentication();
	app.UseAuthorization();

	app.UseEndpoints(endpoints =>
	{
		endpoints.MapControllers();
	});
}
```

## Step 4

Add the [Authorize] attribute to the API controller in the WebApi project

```csharp
[Route("api/[controller]")]
public class ValuesController : Controller
{
	[Authorize]
		
```

## Step 5

Use the secure endpoint in the Razor Page application in the Index page

```csharp
public async Task OnGetAsync()
{
	//var result = await _apiService.GetUnsecureApiDataAsync();
	var resultSecure = await _apiService.GetApiDataAsync();

	Console.WriteLine(resultSecure);
}
```

## Step 6

Run the applications in IIS Express. Start all projects in Visual Studio.