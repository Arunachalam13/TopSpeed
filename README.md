# TopSpeed

# ASP.NET Core MVC Interview Notes

## Brand Management Module (CRUD with Image Upload)

### Overview
This module demonstrates a full CRUD (Create, Read, Update, Delete) operation using ASP.NET Core MVC with EF Core and file/image upload handling. The entity involved is `Brand`, which contains a name, an establishment year, and a brand logo image.

---

## 1. **Brand Model**
```csharp
public class Brand
{
    [Key]
    public Guid Id { get; set; }

    [Required]
    public string Name { get; set; }

    [Display(Name = "Establised Year")]
    public int EstablishedYear { get; set; }

    [Display(Name = "Brand Logo")]
    public string BrandLogo { get; set; }
}
```

---

## 2. **DbContext Configuration**
```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options) : base(options) { }

    public DbSet<Brand> Brands { get; set; }
}
```

---

## 3. **Connection String (appsettings.json)**
```json
"ConnectionStrings": {
  "DefaultConnection": "Server=(local);Database=TopSpeedAutomobileV3;Trusted_Connection=True;TrustServerCertificate=True"
}
```

---

## 4. **BrandController.cs Summary**
- Handles all Brand CRUD operations and file uploads.
- Saves uploaded brand logo images to `wwwroot/images/brand/`
- Deletes old image on update or delete.

### Endpoints:
- `Index` - Displays all brands.
- `Create (GET)` - Shows form.
- `Create (POST)` - Handles brand creation and image upload.
- `Details` - View brand info.
- `Edit (GET/POST)` - Edits info and replaces old image.
- `Delete (GET/POST)` - Removes brand and logo from file system.

---

## 5. **Views**

### Index.cshtml
- Lists brands in a grid.
- If no records: displays a message.
- Button to navigate to Create.

### Create.cshtml
- Form with `Name`, `EstablishedYear`, and file upload input.
- Posts form with image using `multipart/form-data`.

### Edit.cshtml
- Pre-fills form with current values.
- Shows current logo preview.
- Allows changing image file.

### Delete.cshtml
- Shows brand details (disabled inputs).
- Shows brand image.
- Confirms delete.

### Details.cshtml
- Displays all brand info.
- Includes image preview.

---

## 6. **Important Concepts Covered**
- MVC architecture
- Model binding
- EF Core migrations
- File upload in ASP.NET Core
- Serving static files
- Using IWebHostEnvironment to get path to wwwroot
- Image path storage in DB
- Conditional rendering in Razor

---

## 7. **N-Tier Architecture in ASP.NET Core MVC**

### What is N-Tier Architecture?

N-tier architecture (also called layered architecture) is a software design pattern that separates an application into distinct logical layers (or tiers) to improve separation of concerns, maintainability, testability, and scalability.

### Common Layers:

1. **Presentation Layer (UI)**

   - ASP.NET Core MVC Controllers and Views.
   - Handles user interface and inputs.

2. **Business Logic Layer (BLL or Services)**

   - Contains core business logic.
   - Processes input and applies rules before calling the data access layer.

3. **Data Access Layer (DAL)**

   - Responsible for interacting with the database (usually via EF Core).
   - Handles all data-related operations.

4. **Entity Layer (Models)**

   - Contains the entity classes used across layers.
   - Example: `Brand.cs`

### Benefits:

- Loose coupling between components.
- Easier to test and debug.
- Clear separation of responsibilities.
- Enhances code reusability and scalability.

### Example Structure:

```
TopSpeed.Web           (Presentation Layer)
TopSpeed.Services      (Business Logic Layer)
TopSpeed.DataAccess    (Data Access Layer)
TopSpeed.Models        (Entities)
```

Each layer depends only on the one directly beneath it, and never skips layers.


## 8. **Repository Pattern in ASP.NET Core MVC**

### Purpose
- Abstracts data access logic
- Provides a clean separation between business logic and persistence
- Helps achieve a loosely coupled and testable design

### Key Interfaces and Classes

**IGenericRepository<T>** - common CRUD operations:
```csharp
Task Create(T entity);
Task Delete(T entity);
Task<List<T>> Get(Expression<Func<T, bool>> predicate);
Task<List<T>> GetAllAsync();
Task<T> GetByIdAsync(Guid id);
Task<bool> IsRecordExists(Expression<Func<T, bool>> predicate);
```

**IBrandRepository** - specific brand operations:
```csharp
Task Update(Brand brand);
```

**GenericRepository<T>** - implements generic repository:
- Uses EF Core DbSet to implement `IGenericRepository<T>`
- Supports LINQ, async/await

**BrandRepository** - inherits GenericRepository:
- Adds update logic specific to `Brand`

---

## 9. **Unit of Work Pattern in ASP.NET Core MVC**

### What is Unit of Work?
- A design pattern to maintain data consistency across multiple repository operations
- Groups related operations into a single transaction scope
- Ensures `SaveChanges` is called only once per business operation

### Interfaces and Implementation:

**IUnitOfWork**
```csharp
public interface IUnitOfWork : IDisposable
{
    public IBrandRepository Brand { get; }
    Task SaveAsync();
}
```

**UnitOfWork.cs**
```csharp
public class UnitOfWork : IUnitOfWork
{
    protected readonly ApplicationDbContext _dbContext;

    public UnitOfWork(ApplicationDbContext dbContext)
    {
        _dbContext = dbContext;
        Brand = new BrandRepository(_dbContext);
    }

    public IBrandRepository Brand { get; private set; }

    public async Task SaveAsync()
    {
        await _dbContext.SaveChangesAsync();
    }

    public void Dispose()
    {
        _dbContext.Dispose();
    }
}
```

### Benefits
- Keeps repository operations organized
- Ensures consistent commit or rollback
- Simplifies service logic with centralized save method

---

Ready for next steps: service interfaces, dependency injection, or business rule implementation.

