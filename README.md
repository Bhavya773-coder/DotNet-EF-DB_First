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
  - Author.cs
```

## Step 4: Register DbContext in `Program.cs`
```csharp
builder.Services.AddDbContext<DataBaseNameContext>();
```

## Step 5: Create a Controller
```csharp
using Projectname.Models;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore.Metadata;

namespace Lib_API.Controllers
{
    [Route("/api/[controller]")]
    [ApiController]
    public class AuthorController : ControllerBase
    {
        
        private readonly LibManagementContext _context;
    
        public AuthorController(LibManagementContext context)
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
        public async Task<IActionResult> Add([FromBody] Author? entity)
        {
            if (entity == null)
            {
                return BadRequest(new { Message = "Invalid entity data." });
            }

            await _context.Authors.AddAsync(entity);
            await _context.SaveChangesAsync();

            return CreatedAtAction(nameof(GetById), new { id = entity.AuthorId }, entity);
        }
        #endregion
        
        #region Update
        [HttpPut("{id}")]
        public async Task<IActionResult> Edit(int id, [FromBody] Author? updatedEntity)
        {
            if (updatedEntity == null || id != updatedEntity.AuthorId)
            {
                return BadRequest(new { Message = "Invalid entity data or mismatched ID." });
            }

            var existingEntity = await _context.Authors.FindAsync(id);

            if (existingEntity == null)
            {
                return NotFound(new { Message = $"Entity with ID {id} not found." });
            }

            // Update the properties of the existing entity
            existingEntity.AuthorName = updatedEntity.AuthorName;
            existingEntity.AuthorRating= updatedEntity.AuthorRating;
            existingEntity.NumOfBooks = updatedEntity.NumOfBooks;
            // Add more fields as needed

            try
            {
                await _context.SaveChangesAsync();
                return NoContent(); // Successful update, no content to return
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
                return BadRequest();
            }
            
        }
        #endregion
        
    }
}

```




