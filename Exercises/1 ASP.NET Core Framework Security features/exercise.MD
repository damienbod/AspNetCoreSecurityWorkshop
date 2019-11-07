
# Adding FIDO2 passwordless to an ASP.NET Core Identity project

Test user : alice@alice.ch
Test user : bob@bob.ch

No password is required.

## Exercise Goal

The aim of this exerise is to learn how to implement a FIDO2 passwordless authentication with ASP.NET Core and ASP.NET Core Identity.

## Description

In this exercise, we want to use FIDO2 passwordless to sign in in an ASP.NET Core Identity project.

The OSS project FIDO2 is used to implement to FIDO2 logic. This is already added.


```csharp
<PackageReference Include="Fido2" Version="1.0.1" />
```

The code to implement to FIDO2 register and sign in is already added and ready to use. 
The C# code is completely in the FIDO2 folder and the javascript is added to the wwwroot/js.

The FIDO2 persistent is added using EntityFramework Core. This is already added to the ApplicationDbContext

```csharp
public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
    : base(options)
{
}

public DbSet<FidoStoredCredential> FidoStoredCredential { get; set; }

protected override void OnModelCreating(ModelBuilder builder)
{
    builder.Entity<FidoStoredCredential>().HasKey(m => m.Username);

    base.OnModelCreating(builder);
}
```
The exercise is to take the FIDO2 code and use this to implement the Login and Register.

### Step 1 

Scaffold in the Login and the Register Identity Razor Pages to the project.

### Step 2 

Remove all the logic from the Login.cshtml.cs. Replace this with an empty Page.

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace AspNetCoreIdentityFido2Passwordless.Areas.Identity.Pages.Account
{
    [AllowAnonymous]
    public class LoginModel : PageModel
    {
        public void OnGet()
        {
        }

        public void OnPost()
        {
        }
    }
}

```

### Step 3

Add the html code which calls and uses the FIDO2 javascript logic for login implement in the passwordless.login.js.

```html
@page
@model LoginModel

@{
    ViewData["Title"] = "Log in";
}

<h1>@ViewData["Title"]</h1>
<div class="row">
    <div class="col-md-4">
        <section>
            <form action="/mfa" method="post" id="signin">
                <div class="form-group">
                    <label name="username">Email</label>
                    <input name="username" class="form-control" />
                </div>

                <div class="field">
                    <div class="control">
                        <button class="btn btn-primary">Login</button>
                    </div>
                </div>
            </form>
        </section>
    </div>

</div>

<script src="~/js/helpers.js"></script>
<script src="~/js/instant.js"></script>
<script src="~/js/passwordless.login.js"></script>

```

### Step 4

Remove all the logic from the Register.cshtml.cs. Replace this with an empty Page.

```csharp
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace AspNetCoreIdentityFido2Passwordless.Areas.Identity.Pages.Account
{
    [AllowAnonymous]
    public class RegisterModel : PageModel
    {
        public void OnGet()
        {
        }

        public void OnPost()
        {
        }
    }
}

```

### Step 5

Add the html code which calls and uses the FIDO2 javascript logic for register implement in the passwordless.register.js.

```html
@page
@{
    ViewData["Title"] = "Register";
}

<h1>@ViewData["Title"]</h1>

<div class="row">
    <div class="col-md-4">
        <form action="/mfa" method="post" id="register">
            <div class="form-group">
                <label name="username">Email</label>
                <input name="username" class="form-control" />
            </div>

            <div class="form-group">
                <label name="displayName">Display name</label>
                <input name="displayName" class="form-control" />
            </div>

            <div class="field">
                <div class="control">
                    <button class="btn btn-primary">Register user</button>
                </div>
            </div>
        </form>
    </div>
</div>

<script src="~/js/helpers.js"></script>
<script src="~/js/instant.js"></script>
<script src="~/js/passwordless.register.js"></script>


```

### Step 6

Add the FIDO2 sevices and configurations to the ConfigureServices

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlite(Configuration.GetConnectionString("DefaultConnection")));
    services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true)
        .AddEntityFrameworkStores<ApplicationDbContext>();

    services.AddControllers()
        .AddNewtonsoftJson();

    services.AddRazorPages();

    services.Configure<Fido2Configuration>(Configuration.GetSection("fido2"));
    services.Configure<Fido2MdsConfiguration>(Configuration.GetSection("fido2mds"));
    services.AddScoped<Fido2Storage>();
    // Adds a default in-memory implementation of IDistributedCache.
    services.AddDistributedMemoryCache();
    services.AddSession(options =>
    {
        // Set a short timeout for easy testing.
        options.IdleTimeout = TimeSpan.FromMinutes(2);
        options.Cookie.HttpOnly = true;
        options.Cookie.SameSite = Microsoft.AspNetCore.Http.SameSiteMode.None;
    });
}
```

### Step 7

Add the app.UseSession(); to the Configure method in the Startup class

```csharp
app.UseRouting();

app.UseAuthentication();
app.UseAuthorization();

app.UseSession();

app.UseEndpoints(endpoints =>
{
    endpoints.MapRazorPages();
    endpoints.MapControllers();
});

```

## NOTE

The FIDO2 services require the following app settings which would be set per deployment as required
```json
  "fido2": {
    "serverDomain": "localhost",
    "serverName": "Fido2PasswordlessTest",
    "origin": "https://localhost:44326",
    "timestampDriftTolerance": 300000
  },
  "fido2mds": {
    "MDSAccessKey": null
  },
```