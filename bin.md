# CRUD Implementation - GradeSense API


## 📊 Tables Priority & Complexity

| Table | Priority | Complexity | Dependencies | Status |
|-------|----------|------------|--------------|--------|
| **Users** | 1 | Medium | None | 95% → 100% |
| **Departments** | 2 | Low | Users (HOD) | 95% → 100% |
| **Faculties** | 3 | Medium | Users + Departments | 0% → 100% |
| **Students** | 4 | Medium | Users + Departments | 0% → 100% |
| Batches | 5 | Medium | Departments + Faculties | Later |
| Subjects | 6 | Low | Departments | Later |
| SubjectUnits | 7 | Low | Subjects | Later |
| CourseOfferings | 8 | High | Multiple | Later |
| CourseEnrollments | 9 | High | Multiple | Later |
| EvaluationSchemes | 10 | Medium | CourseOfferings | Later |
| AssessmentItems | 11 | Medium | EvaluationSchemes | Later |
| StudentMarks | 12 | Medium | Multiple | Later |
| FacultyAssignments | 13 | Low | CourseOfferings + Faculties | Later |
| AttendanceRecords | 14 | Low | CourseEnrollments | Later |
| UploadHistory | 15 | Low | Multiple | Later |
| AuditLog | 16 | Low | Users | Later |
| Predictions | 17 | Low | CourseEnrollments | Later |

---

## 🏗️ Standard CRUD Structure (Template)

For each entity, we'll create:

### Folder Structure
```
DTOs/
  {EntityName}/
    Request/
      - Create{EntityName}Request.cs
      - Update{EntityName}Request.cs
      - {EntityName}FilterRequest.cs
    Response/
      - {EntityName}Response.cs
      - {EntityName}DetailResponse.cs
      - {EntityName}ListResponse.cs

Validators/                          ← NEW FOLDER
  {EntityName}/
    - Create{EntityName}RequestValidator.cs
    - Update{EntityName}RequestValidator.cs

Interfaces/
  Repositories/
    - I{EntityName}Repository.cs
  Services/
    - I{EntityName}Service.cs

Repositories/
  - {EntityName}Repository.cs

Services/
  - {EntityName}Service.cs

Controllers/
  - {EntityName}Controller.cs
```

### Standard CRUD Operations

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/{entity}` | Get all (with pagination & filters) |
| GET | `/api/{entity}/{id}` | Get by ID |
| POST | `/api/{entity}` | Create new |
| PUT | `/api/{entity}/{id}` | Update existing |
| DELETE | `/api/{entity}/{id}` | Soft delete |

---

## 🔧 FluentValidation Benefits

### Why FluentValidation over Data Annotations?

| Feature | Data Annotations | FluentValidation |
|---------|------------------|------------------|
| **Separation of Concerns** | ❌ Mixed with DTOs | ✅ Separate files |
| **Complex Rules** | ❌ Limited | ✅ Very flexible |
| **Conditional Validation** | ❌ Hard | ✅ Easy (.When()) |
| **Reusability** | ❌ Duplicate code | ✅ Reusable rules |
| **Testability** | ❌ Hard to test | ✅ Easy to test |
| **Custom Messages** | ⚠️ Basic | ✅ Advanced |
| **Async Validation** | ❌ Not supported | ✅ Supported |
| **Dependent Properties** | ❌ Complex | ✅ Simple |

### Example Comparison

**Data Annotations:**
```csharp
public class CreateUserRequest
{
    [Required(ErrorMessage = "Email is required")]
    [EmailAddress(ErrorMessage = "Invalid email")]
    [StringLength(255)]
    public string Email { get; set; }
    
    [Required]
    [StringLength(100, MinimumLength = 6)]
    public string Password { get; set; }
}
```

**FluentValidation:**
```csharp
public class CreateUserRequestValidator : AbstractValidator<CreateUserRequest>
{
    public CreateUserRequestValidator()
    {
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format")
            .MaximumLength(255).WithMessage("Email cannot exceed 255 characters")
            .MustAsync(BeUniqueEmail).WithMessage("Email already exists");
        
        RuleFor(x => x.Password)
            .NotEmpty().WithMessage("Password is required")
            .MinimumLength(6).WithMessage("Password must be at least 6 characters")
            .Matches(@"[A-Z]").WithMessage("Password must contain uppercase letter")
            .Matches(@"[a-z]").WithMessage("Password must contain lowercase letter")
            .Matches(@"[0-9]").WithMessage("Password must contain number")
            .Matches(@"[@$!%*?&#]").WithMessage("Password must contain special character");
    }

}
```
Remember: FluentValidation throws AsyncValidatorInvokedSynchronouslyException if async methods are called synchronously.

---

## 📝 Implementation Checklist

### Phase 1: Review User CRUD with FluentValidation

### Phase 2: Review Departments CRUD

### Phase 3: Faculties CRUD
- [ ] Create DTOs
- [ ] Create Validators
- [ ] Create Repository
- [ ] Create Service (handle User creation)
- [ ] Create Controller
- [ ] Test all endpoints

### Phase 4: Students CRUD
- [ ] Create DTOs
- [ ] Create Validators
- [ ] Create Repository
- [ ] Create Service (handle User creation)
- [ ] Create Controller
- [ ] Test all endpoints

then we'll move to Batches, Subjects, etc.

---

## 🎯 Features for Each CRUD

### Standard Features
1. ✅ **Pagination** - PageNumber, PageSize
2. ✅ **Filtering** - Search, Status filters
3. ✅ **Sorting** - SortBy, SortOrder
4. ✅ **Soft Delete** - DeletedAt timestamp
5. ✅ **Validation** - FluentValidation
6. ✅ **Authorization** - Role-based access
7. ✅ **Error Handling** - Consistent format
8. ✅ **Logging** - Track all operations

### Entity-Specific Features

#### Departments
- Check HOD is a Faculty user
- Can't delete if has faculties/students
- Can't delete if has subjects/batches

#### Faculties
- Must have valid User ID
- Create User automatically if needed
- Can't delete if teaching courses
- Can't delete if is HOD of department

#### Students
- Must have valid User ID
- Create User automatically if needed
- Can't delete if has enrollments
- Can't delete if has grades

---

## 🔐 Authorization Rules

| Entity | Create | Read | Update | Delete |
|--------|--------|------|--------|--------|
| **Users** | Admin | Any Auth | Admin/Self | Admin |
| **Departments** | Admin | Any Auth | Admin | Admin |
| **Faculties** | Admin | Any Auth | Admin/Self | Admin |
| **Students** | Admin | Faculty/Admin | Admin | Admin |

---

## 📊 Validation Rules Summary

### Users
- Email: Required, valid format, unique
- Password: Required, min 6 chars, complexity rules
- FullName: Required, 2-255 chars
- Role: Required, must be Student/Faculty/Admin

### Departments
- Name: Required, unique, 2-255 chars
- Code: Required, unique, 2-10 chars, uppercase
- HODUserId: Optional, must be Faculty user if provided

### Faculties
- UserId: Required, must be valid User with Role=Faculty
- EmployeeId: Required, unique
- DepartmentId: Required, must exist
- Designation: Optional, predefined list
- JoiningDate: Optional, cannot be future date

### Students
- UserId: Required, must be valid User with Role=Student
- EnrollmentNumber: Required, unique
- AdmissionYear: Required, 2000-current year
- CurrentSemester: Required, 1-8
- DepartmentId: Required, must exist
- CGPA: Optional, 0-10

---

## 🚀 Performance Optimizations

1. **Eager Loading** - Include related entities where needed
2. **Projection** - Select only needed fields
3. **Caching** - Cache reference data (departments, subjects)
4. **Indexing** - Database indexes on foreign keys
5. **Async/Await** - All database operations async
6. **Pagination** - Always paginate list endpoints

---

Ready to implement? Let's start! 🚀