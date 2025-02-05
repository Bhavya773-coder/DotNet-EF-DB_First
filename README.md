# README.md

## Dependencies
- `Microsoft.EntityFrameworkCore.Design`
- `Microsoft.EntityFrameworkCore.SqlServer`
- `Microsoft.EntityFrameworkCore.Tools`
- `Microsoft.EntityFrameworkCore.SqlLite`

## Step 1: Create Table
```sql
CREATE TABLE [dbo].[Author](
    [author_id] [int] IDENTITY(1,1) NOT NULL,
    [author_name] [varchar](50) NULL,
    [num_of_books] [int] NULL,
    [author_rating] [varchar](50) NULL
);
```

## Step 2: Run Scaffold Command
```sh
# Basic scaffold command
 dotnet ef dbcontext scaffold "Your_Connection_String" Microsoft.EntityFrameworkCore.SqlServer -o Models

# If specifying a project
 dotnet ef dbcontext scaffold "Your_Connection_String" Microsoft.EntityFrameworkCore.SqlServer -o Models --project Your_Project_Name
```

## Step 3: Check the Project Structure
After running the command, the following folder and files will be created:
```
- Models/
  - DatabaseNameContext.cs
  - TableName.cs
```

## Step 4: Register DbContext in `Program.cs`
```csharp
builder.Services.AddDbContext<DataBaseNameContext>();
```

## Step 5: Create a Controller
```csharp
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;

namespace Lib_API.Controllers
{
    [Route("/api/[controller]")]
    [ApiController]
    public class AuthorController : ControllerBase
    {
        private readonly DataBaseNameContext _context;
    
        public AuthorController(DataBaseNameContext context)
        {
            _context = context;
        }

        #region GetAll
        [HttpGet]
        public async Task<ActionResult> GetApi()
        {
            return Ok(await _context.Authors.ToListAsync());
        }
        #endregion

        #region GetById
        [HttpGet("{id}")]
        public async Task<IActionResult> GetById(int id)
        {
            var entity = await _context.Authors.FindAsync(id);
            if (entity == null)
            {
                return NotFound(new { Message = $"Entity with ID {id} not found." });
            }
            return Ok(entity);
        }
        #endregion

        #region Create
        [HttpPost]
        public async Task<IActionResult> Add([FromBody] Author entity)
        {
            if (entity == null)
            {
                return BadRequest(new { Message = "Invalid entity data." });
            }
            await _context.Authors.AddAsync(entity);
            await _context.SaveChangesAsync();
            return CreatedAtAction(nameof(GetById), new { id = entity.author_id }, entity);
        }
        #endregion

        #region Update
        [HttpPut("{id}")]
        public async Task<IActionResult> Edit(int id, [FromBody] Author updatedEntity)
        {
            if (updatedEntity == null || id != updatedEntity.author_id)
            {
                return BadRequest(new { Message = "Invalid entity data or mismatched ID." });
            }
            var existingEntity = await _context.Authors.FindAsync(id);
            if (existingEntity == null)
            {
                return NotFound(new { Message = $"Entity with ID {id} not found." });
            }
            // Update fields
            existingEntity.author_name = updatedEntity.author_name;
            existingEntity.num_of_books = updatedEntity.num_of_books;
            existingEntity.author_rating = updatedEntity.author_rating;
            
            try
            {
                await _context.SaveChangesAsync();
                return NoContent();
            }
            catch (DbUpdateException ex)
            {
                return StatusCode(500, new { Message = "An error occurred while updating the entity.", Details = ex.Message });
            }
        }
        #endregion

        #region Delete
        [HttpDelete("{id}")]
        public async Task<IActionResult> Delete(int id)
        {
            var entity = await _context.Authors.FindAsync(id);
            if (entity == null)
            {
                return NotFound(new { Message = $"Entity with ID {id} not found." });
            }
            _context.Authors.Remove(entity);
            try
            {
                await _context.SaveChangesAsync();
                return NoContent();
            }
            catch (Exception e)
            {
                return BadRequest(new { Message = "Error deleting entity.", Details = e.Message });
            }
        }
        #endregion
    }
}
```

### Changes and Improvements:
- Fixed class name inconsistencies (`YourController` → `AuthorController`).
- Used `Authors` instead of `TableName` for DbSet.
- Updated variable names to match `Author` model.
- Proper error handling and status codes.

Let me know if you need more modifications! 🚀

