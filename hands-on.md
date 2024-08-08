Sure! Here's a guide to creating a CRUD application using ASP.NET Core Razor Pages.

### Step 1: Set Up the ASP.NET Core Project

1. **Create a new ASP.NET Core Razor Pages project**:
   ```sh
   dotnet new webapp -n MyRazorCrudApp
   cd MyRazorCrudApp
   ```

2. **Add EF Core and a database provider**:
   ```sh
   dotnet add package Microsoft.EntityFrameworkCore.SqlServer
   dotnet add package Microsoft.EntityFrameworkCore.Tools
   ```

### Step 2: Create the Model

1. **Create a new model class**. For this example, let's create a `Product` class.

   ```csharp
   public class Product
   {
       public int Id { get; set; }
       public string Name { get; set; }
       public decimal Price { get; set; }
   }
   ```

### Step 3: Set Up the Database Context

1. **Create a `DataContext` class**:

   ```csharp
   using Microsoft.EntityFrameworkCore;

   public class DataContext : DbContext
   {
       public DataContext(DbContextOptions<DataContext> options) : base(options) { }

       public DbSet<Product> Products { get; set; }
   }
   ```

2. **Register the context in `Program.cs`**:

   ```csharp
   var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddRazorPages();

builder.Services.AddDbContext<DataContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapRazorPages();

app.Run();
   ```

3. **Add the connection string to `appsettings.json`**:

   ```json
   "ConnectionStrings": {
       "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=MyRazorCrudAppDb;Trusted_Connection=True;"
   }
   ```

### Step 4: Create the Razor Pages

1. **Create a folder named `Pages/Products`**.

2. **Add the following Razor Pages and their respective code-behind files**:

#### Index (List of Products)
- **Pages/Products/Index.cshtml**:
   ```html
   @page
   @model MyRazorCrudApp.Pages.Products.IndexModel

   <h1>Products</h1>

   <a asp-page="Create">Create New</a>

   <table class="table">
       <thead>
           <tr>
               <th>Name</th>
               <th>Price</th>
               <th></th>
           </tr>
       </thead>
       <tbody>
           @foreach (var item in Model.Products)
           {
               <tr>
                   <td>@item.Name</td>
                   <td>@item.Price</td>
                   <td>
                       <a asp-page="Edit" asp-route-id="@item.Id">Edit</a> |
                       <a asp-page="Details" asp-route-id="@item.Id">Details</a> |
                       <a asp-page="Delete" asp-route-id="@item.Id">Delete</a>
                   </td>
               </tr>
           }
       </tbody>
   </table>
   ```

- **Pages/Products/Index.cshtml.cs**:
   ```csharp
   using Microsoft.AspNetCore.Mvc.RazorPages;
   using Microsoft.EntityFrameworkCore;
   using System.Collections.Generic;
   using System.Threading.Tasks;

   public class IndexModel : PageModel
   {
       private readonly DataContext _context;

       public IndexModel(DataContext context)
       {
           _context = context;
       }

       public IList<Product> Products { get; set; }

       public async Task OnGetAsync()
       {
           Products = await _context.Products.ToListAsync();
       }
   }
   ```

#### Create
- **Pages/Products/Create.cshtml**:
   ```html
   @page
   @model MyRazorCrudApp.Pages.Products.CreateModel

   <h1>Create Product</h1>

   <form method="post">
       <div class="form-group">
           <label>Name</label>
           <input type="text" class="form-control" asp-for="Product.Name" />
       </div>
       <div class="form-group">
           <label>Price</label>
           <input type="text" class="form-control" asp-for="Product.Price" />
       </div>
       <button type="submit" class="btn btn-primary">Save</button>
       <a asp-page="Index">Cancel</a>
   </form>
   ```

- **Pages/Products/Create.cshtml.cs**:
   ```csharp
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.AspNetCore.Mvc.RazorPages;
   using System.Threading.Tasks;

   public class CreateModel : PageModel
   {
       private readonly DataContext _context;

       public CreateModel(DataContext context)
       {
           _context = context;
       }

       [BindProperty]
       public Product Product { get; set; }

       public IActionResult OnGet()
       {
           return Page();
       }

       public async Task<IActionResult> OnPostAsync()
       {
           if (!ModelState.IsValid)
           {
               return Page();
           }

           _context.Products.Add(Product);
           await _context.SaveChangesAsync();

           return RedirectToPage("./Index");
       }
   }
   ```

#### Edit
- **Pages/Products/Edit.cshtml**:
   ```html
   @page
   @model MyRazorCrudApp.Pages.Products.EditModel

   <h1>Edit Product</h1>

   <form method="post">
       <input type="hidden" asp-for="Product.Id" />
       <div class="form-group">
           <label>Name</label>
           <input type="text" class="form-control" asp-for="Product.Name" />
       </div>
       <div class="form-group">
           <label>Price</label>
           <input type="text" class="form-control" asp-for="Product.Price" />
       </div>
       <button type="submit" class="btn btn-primary">Save</button>
       <a asp-page="Index">Cancel</a>
   </form>
   ```

- **Pages/Products/Edit.cshtml.cs**:
   ```csharp
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.AspNetCore.Mvc.RazorPages;
   using Microsoft.EntityFrameworkCore;
   using System.Threading.Tasks;

   public class EditModel : PageModel
   {
       private readonly DataContext _context;

       public EditModel(DataContext context)
       {
           _context = context;
       }

       [BindProperty]
       public Product Product { get; set; }

       public async Task<IActionResult> OnGetAsync(int id)
       {
           Product = await _context.Products.FindAsync(id);

           if (Product == null)
           {
               return NotFound();
           }

           return Page();
       }

       public async Task<IActionResult> OnPostAsync()
       {
           if (!ModelState.IsValid)
           {
               return Page();
           }

           _context.Attach(Product).State = EntityState.Modified;

           try
           {
               await _context.SaveChangesAsync();
           }
           catch (DbUpdateConcurrencyException)
           {
               if (!_context.Products.Any(e => e.Id == Product.Id))
               {
                   return NotFound();
               }
               else
               {
                   throw;
               }
           }

           return RedirectToPage("./Index");
       }
   }
   ```

#### Delete
- **Pages/Products/Delete.cshtml**:
   ```html
   @page
   @model MyRazorCrudApp.Pages.Products.DeleteModel

   <h1>Delete Product</h1>

   <h3>Are you sure you want to delete this?</h3>
   <div>
       <h4>Product</h4>
       <hr />
       <dl class="row">
           <dt class="col-sm-2">Name</dt>
           <dd class="col-sm-10">@Model.Product.Name</dd>
           <dt class="col-sm-2">Price</dt>
           <dd class="col-sm-10">@Model.Product.Price</dd>
       </dl>

       <form method="post">
           <input type="hidden" asp-for="Product.Id" />
           <button type="submit" class="btn btn-danger">Delete</button>
           <a asp-page="Index">Cancel</a>
       </form>
   </div>
   ```

- **Pages/Products/Delete.cshtml.cs**:
   ```csharp
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.AspNetCore.Mvc.RazorPages;
   using System.Threading.Tasks;

   public class DeleteModel : PageModel
   {
       private readonly DataContext _context;

       public DeleteModel(DataContext context)
       {
           _context = context;
       }

       [BindProperty]
       public Product Product { get; set; }

       public async Task<IActionResult> OnGetAsync(int id)
       {
           Product = await _context.Products.FindAsync(id);

           if (Product == null)
           {
               return NotFound();
           }

           return Page();
       }

       public async Task<IActionResult> OnPostAsync(int id)
       {
           Product = await _context.Products.FindAsync(id);

           if (Product != null)
           {
               _context.Products.Remove(Product);
               await _context.SaveChangesAsync();
           }

           return RedirectToPage("./Index

");
       }
   }
   ```

#### Details
- **Pages/Products/Details.cshtml**:
   ```html
   @page
   @model MyRazorCrudApp.Pages.Products.DetailsModel

   <h1>Product Details</h1>

   <div>
       <h4>Product</h4>
       <hr />
       <dl class="row">
           <dt class="col-sm-2">Name</dt>
           <dd class="col-sm-10">@Model.Product.Name</dd>
           <dt class="col-sm-2">Price</dt>
           <dd class="col-sm-10">@Model.Product.Price</dd>
       </dl>
   </div>

   <a asp-page="Index">Back to List</a>
   ```

- **Pages/Products/Details.cshtml.cs**:
   ```csharp
   using Microsoft.AspNetCore.Mvc;
   using Microsoft.AspNetCore.Mvc.RazorPages;
   using System.Threading.Tasks;

   public class DetailsModel : PageModel
   {
       private readonly DataContext _context;

       public DetailsModel(DataContext context)
       {
           _context = context;
       }

       public Product Product { get; set; }

       public async Task<IActionResult> OnGetAsync(int id)
       {
           Product = await _context.Products.FindAsync(id);

           if (Product == null)
           {
               return NotFound();
           }

           return Page();
       }
   }
   ```

### Step 5: Migrate the Database

1. **Add the initial migration and update the database**:

   ```sh
   dotnet ef migrations add InitialCreate
   dotnet ef database update
   ```

### Step 6: Run the Application

1. **Run the application**:

   ```sh
   dotnet run
   ```

This setup provides a basic CRUD application using Razor Pages. You can enhance it further with features like validation, authentication, authorization, and more.
