
### Step 1: Install Required NuGet Packages

Ensure you have the necessary NuGet packages installed. You need `Microsoft.EntityFrameworkCore.SqlServer` and `Microsoft.EntityFrameworkCore.Tools`.

You can install them via the NuGet Package Manager or using the Package Manager Console:

```bash
Install-Package Microsoft.EntityFrameworkCore.SqlServer
Install-Package Microsoft.EntityFrameworkCore.Tools
```

### Step 2: Create the Database Context

Create a database context class that derives from `DbContext`. This class represents a session with the database, allowing you to query and save data.

#### Example `ApplicationDbContext.cs`:

```csharp
using Microsoft.EntityFrameworkCore;
using YourNamespace.Models; // Replace with your actual namespace

namespace YourNamespace.Data
{
    public class ApplicationDbContext : DbContext
    {
        public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
            : base(options)
        {
        }

        public DbSet<Product> Products { get; set; }
        // Add other DbSet properties as needed
    }
}
```

### Step 3: Configure the Connection String

Add a connection string to your `appsettings.json` file. This file typically resides in the root of your project.

#### Example `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\MSSQLLocalDB;Database=YourDatabaseName;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

Replace `YourDatabaseName` with the name you want for your database.

### Step 4: Configure Services in `Startup.cs`

Configure Entity Framework Core to use the connection string in the `Startup.cs` file.

#### Example `Startup.cs`:

```csharp
public class Startup
{
    public Startup(IConfiguration configuration)
    {
        Configuration = configuration;
    }

    public IConfiguration Configuration { get; }

    public void ConfigureServices(IServiceCollection services)
    {
        services.AddRazorPages();

        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(
                Configuration.GetConnectionString("DefaultConnection")));
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
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

        app.UseAuthorization();

        app.UseEndpoints(endpoints =>
        {
            endpoints.MapRazorPages();
        });
    }
}
```

### Step 5: Create a Model

Create a model class that represents the data structure you want to work with.

#### Example `Product.cs`:

```csharp
using System.ComponentModel.DataAnnotations;

namespace YourNamespace.Models
{
    public class Product
    {
        public int Id { get; set; }

        [Required]
        [StringLength(100)]
        public string Name { get; set; }

        [Required]
        [Range(0.01, 10000)]
        public decimal Price { get; set; }
    }
}
```

### Step 6: Create a Migration and Update the Database

Use the Entity Framework Core tools to create a migration and update the database.

In the Package Manager Console, run:

```bash
Add-Migration InitialCreate
Update-Database
```

This will create the initial database schema based on your model.

### Step 7: Use the DbContext in Razor Pages

Inject the `ApplicationDbContext` into your Razor Pages and use it to interact with the database.

#### Example `Index.cshtml.cs`:

```csharp
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.EntityFrameworkCore;
using YourNamespace.Data;
using YourNamespace.Models;
using System.Collections.Generic;
using System.Threading.Tasks;

namespace YourNamespace.Pages.Products
{
    public class IndexModel : PageModel
    {
        private readonly ApplicationDbContext _context;

        public IndexModel(ApplicationDbContext context)
        {
            _context = context;
        }

        public IList<Product> Products { get; set; }

        public async Task OnGetAsync()
        {
            Products = await _context.Products.ToListAsync();
        }
    }
}
```

#### Example `Index.cshtml`:

```html
@page
@model YourNamespace.Pages.Products.IndexModel

<h2>Products</h2>

<table class="table">
    <thead>
        <tr>
            <th>Name</th>
            <th>Price</th>
        </tr>
    </thead>
    <tbody>
        @foreach (var product in Model.Products)
        {
            <tr>
                <td>@product.Name</td>
                <td>@product.Price</td>
            </tr>
        }
    </tbody>
</table>
```

### Summary

1. Install the necessary NuGet packages.
2. Create the `ApplicationDbContext` class.
3. Add the connection string to `appsettings.json`.
4. Configure the database context in `Startup.cs`.
5. Create your model class.
6. Create and apply migrations.
7. Use the `ApplicationDbContext` in your Razor Pages.

This setup allows you to connect to and interact with `MSSQLLocalDB` using Entity Framework Core in your ASP.NET Core Razor Pages application. If you have any specific issues or questions, feel free to ask!
