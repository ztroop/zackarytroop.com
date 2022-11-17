+++
title = "REST Web API Introduction in C# - Part One"
slug = "csharp-rest-api-intro-part-one"
date = 2020-11-04
+++

# Summary

In this article, we'll be discussing the REST architectural style and demonstrate concepts through
building an **AP**plication **I**nterface (**API**) in C# .NET Core! Exciting stuff, let's get started!

##### Table of Contents
1. [REST Introduction](#rest-introduction)
2. [History](#history)
3. [Platform Agnostic](#platform-agnostic)
4. [Intended Audience](#intended-audience)
5. [Enviroment Setup](#environment-setup)
6. [Installing Dependencies](#installing-dependencies)
7. [Code Walkthrough](#code-walkthrough)

Links referenced are _encouraged_ to be explored for learning reinforcement.

## REST Introduction

**RE**presentational **S**tate **T**ransfer (**REST**) is an software architectural style that defines
a set of constraints to be used for creating software. It was originally proposed by Roy Fielding
in his [dissertation](https://www.ics.uci.edu/~fielding/pubs/dissertation/fielding_dissertation.pdf).
It's important to note that REST is not only applied to the web, via HTTP protocol.
Other protocols like [CoAP](https://tools.ietf.org/html/rfc7252) also employ REST principles.
This architectural style provides conventions that makes development easier for engineers and improves
interoperability between systems.

See [Representation State Transfer](https://en.wikipedia.org/wiki/Representational_state_transfer) for more information.

## History

In the past, developing C# web services typically involved using **S**imple **O**bject **A**ccess
**P**rotocol (**SOAP**), a standards-based web services access protocol that's been around for a long
time. This, along with **W**indows **C**ommunication **F**oundation (**WCF**) are on their way out.

At this point, I think you'll only be interacting with these from a legacy maintenance point of view.
(I can only hope for your sake, anyway!)

REST permits many different data formats including plain text, HTML, XML, and JSON, which yields
better compatibility between systems, unlike SOAP.

## Intended Audience

This article assumes that the reader has an understanding of common programming idioms and patterns in the C# language. Experience with the command prompt or terminal is recommended, but not required. We'll touch on various concepts, but further research might be required if you're just starting out.

## Tooling

We'll be using a terminal (or command prompt) and Visual Code for code editing.
I've found that developers learning C# become too dependent on Visual Studio. In general, I recommend [Visual Code](https://code.visualstudio.com/), as it's an open-source and cross-platform editor.

#### Platform Agnostic

We'll be using .NET Core instead of .NET Framework. The reason is simple, we're focusing on technologies
that are relevant for the future, not the past. See [.NET Core is the Future of .NET](https://devblogs.microsoft.com/dotnet/net-core-is-the-future-of-net/)

> New applications should be built on .NET Core. .NET Core is where future investments in .NET will happen. Existing applications are safe to remain on .NET Framework which will be supported. Existing applications that want to take advantage of the new features in .NET should consider moving to .NET Core.

This has the added benefit of being able to develop on any system; Windows, Mac or Linux!

# Demonstration

Now that we've finished the preamble, let's dig into the activity!
This was developed on Ubuntu Linux, but should be largely transferable to other operating systems.

## Environment

Open up your terminal and create a new project with the commands below.

In Linux:

```sh
mkdir -p ~/projects/CSharpRESTDemo # Creates all directories (nested).
cd ~/projects/CSharpRESTDemo # Change to the top-level directory we created.
dotnet new console # Creates C# project files using the console template.
```

In Windows:

```sh
mkdir "%USERPROFILE%\projects" # Creates the "projects" directory.
cd /d "%USERPROFILE%\projects" # Change to the directory we created.
mkdir CSharpRESTDemo # Creates a directory for our project.
cd CSharpRESTDemo # Change to the directory we just created.
dotnet new console # Creates C# project files using the console template.
```

Why are we using the `console` template instead of `web`? Well, we want to start with a minimal
configuration and build on it throughout this article. In the future though, the `web` template
can save you some time.

We should now see two files and a directory!

1. `CSharpRESTDemo.csproj` is a project file that contains the list of files included in a project along with the references to system assemblies.
2. `Program.cs` is the entry point of our program.
3. `obj` is a directory used to store temporary object files and other files used in order to create the final binary during the compilation process. The `bin` folder is the output folder for complete binaries (or assemblies).

If you're using Microsoft Visual Studio, you will also have a `.sln` extension file. That's a structure file used for organizing projects in Microsoft Visual Studio.

## Installing Dependencies

Before we dig into the code, we'll need to apply some dependencies that are not included in the standard library.
In the terminal, we'll need to be in the working directory of `CSharpRESTDemo` and run the following commands:

```sh
dotnet add package AutoMapper --version 8.0.0
dotnet add package AutoMapper.Extensions.Microsoft.DependencyInjection --version 6.0.0
dotnet add package Microsoft.AspNetCore.App
dotnet add package Microsoft.AspNetCore.JsonPatch --version 3.1.9
dotnet add package Microsoft.AspNetCore.Razor.Design --version 3.1.9
dotnet add package Microsoft.EntityFrameworkCore --version 3.1.9
dotnet add package Microsoft.EntityFrameworkCore.Design --version 3.1.9
dotnet add package Microsoft.EntityFrameworkCore.InMemory --version 3.1.9
dotnet add package Microsoft.EntityFrameworkCore.SqlServer --version 3.1.9
```

1. **AutoMapper** is used in model projection scenarios to flatten complex object models to DTOs and other simple objects.
2. **AspNetCore** provides a framework and tooling for building web apps and services in .NET Core.
3. **EntityFrameworkCore** is a modern **O**bject **R**elational **M**apper (**ORM**) for .NET Core. It supports [LINQ](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/) queries, change tracking, updates, and schema migrations.

## Code Walkthrough

I'll be omitting `using` import lines for the sake of brevity. You can always review the code from the [GitHub repository](https://github.com/ztroop/csharp-rest-api-example).

---

In `Program.cs`, replace contents with the following:

```cs
namespace CSharpRESTDemo
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateWebHostBuilder(args).Build().Run();
        }

        public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>();
    }
}
```

We're creating a host using an instance of `IWebHostBuilder`. This is typically performed in the
app's entry point, the `Main` method as shown above.

When setting up a host, `Configure` and `ConfigureServices` methods can be provided. We specified
our own `Startup` class, which we'll examine next. The `Startup` class _must_ define a `Configure` method.

See [ASP.NET Core Web Host](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/host/web-host?view=aspnetcore-3.1#set-up-a-host) for more information on how this works.

---

Create a new file `Startup.cs` with the following:

```cs
namespace CSharpRESTDemo
{
    public class Startup
    {
        public IConfigurationRoot Configuration { get; }

        public Startup()
        {
            var builder = new ConfigurationBuilder()
                .SetBasePath(Directory.GetCurrentDirectory())
                .AddJsonFile("appsettings.json");
            Configuration = builder.Build();
        }

        public void ConfigureServices(IServiceCollection services)
        {
            services.AddAutoMapper(typeof(AutoMapping));
            services.AddDbContext<DemoDbContext>(
                opt => opt.UseInMemoryDatabase(databaseName: "InMemoryDb")
            );
            services.AddCors(opt => {
                opt.AddPolicy("AllowMyOrigin", builder => builder
                    .WithOrigins("http://localhost:3000")
                    .AllowAnyMethod()
                    .AllowAnyHeader()
                );
            });
            services.AddScoped<ICustomerRepository, CustomerRepository>();
            services.AddMvc(opt => opt.EnableEndpointRouting = false);
        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseCors("AllowMyOrigin");
            app.UseMvc();
        }
    }
}
```

The `Startup.cs` class is where:

1. Services required by the app are configured.
2. The app's request handling pipeline is defined, as a series of middleware components.

There are _two_ methods of interest in this class:

1. `ConfigureServices` is optional and used to configure [dependency injection](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-3.1).
2. `Configure` is used to set up middlewares, routing rules, etc.

In the `Startup` constructor, we explicitly extract configuration values from `appsettings.json`. This configuration pattern is common, but alternatively, the [options pattern](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/options?view=aspnetcore-3.1) _does_ offer a greater separation of concerns.

In the `ConfigureServices` method, we're doing the following:

- **AddAutoMapper** registers AutoMapper and defines a [profile](https://docs.automapper.org/en/stable/Configuration.html#profile-instances). We're using this to map **D**ata **T**ransfer **O**bjects (**DTO**), but more on that later.
- **AddDbContext** registers EntityFramework, where we specify our context and use of a in-memory database.
- **AddCors** registers a CORS policy, which we will reference in the `Configure` method. [CORS](https://docs.microsoft.com/en-us/aspnet/core/security/cors?view=aspnetcore-3.1) is a lengthy topic, but you should read about it if you're unfamiliar with it. Many developers simply bypass it by _allowing any origin_, but this can lead to security problems.
- **AddScoped** registers a scoped lifetime, the [lifetime](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection?view=aspnetcore-3.1#service-lifetimes) of a single request. We'll talk more about _repositories_ later.
- **AddMvc** registers MVC services.

In the `Configure` method, we check to see if the application is running in development mode to determine if we should show the user an exception page if something goes wrong. You wouldn't want to show a stack trace to the user in a production environment. After that, we tell our application to add MVC to the request execution pipeline. MVC will allow us to start defining routes and controllers.

See [ASP.NET Core Fundamentals](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/?view=aspnetcore-3.1&tabs=linux) for more information.

---

Create a new folder called `Entities` and create a new file `DemoDbContext.cs` with the following:

```cs
namespace CSharpRESTDemo.Entities
{
    public class DemoDbContext : DbContext
    {
        public DemoDbContext(DbContextOptions<DemoDbContext> options) : base(options)
        {

        }

        public DbSet<Customer> Customers { get; set; }
    }
}
```

We create a `DbSet` for `Customers`. A `DbSet` provides a mapping to the database table object.

---

In the `Entities` folder, create a file `Customer.cs` with the following:

```cs
namespace CSharpRESTDemo.Entities
{
    public class Customer
    {
        public Guid Id { get; set; }
        public string Firstname { get; set; }
        public string Lastname { get; set; }
        public int Age { get; set; }
    }
}
```

We define a `Customer` entity with _four_ properties. We're using `Guid` in this example to demonstrate
that using an `int` is not always an appropriate choice. You wouldn't want to allow for the possibility
of users iterating over your records in some cases.

---

Next, we'll create a new directory called `Repositories` and create a file `CustomerRepository.cs` with the following:

```cs
namespace CSharpRESTDemo.Repositories
{
    public class CustomerRepository : ICustomerRepository
    {
        private DemoDbContext _context;
        public CustomerRepository(DemoDbContext context)
        {
            _context = context;
        }

        public IQueryable<Customer> GetAll()
        {
            return _context.Customers;
        }

        public Customer GetSingle(Guid id)
        {
            return _context.Customers.FirstOrDefault(x => x.Id == id);
        }

        public void Add(Customer item)
        {
            _context.Customers.Add(item);
        }

        public void Delete(Guid id)
        {
            Customer Customer = GetSingle(id);
            _context.Customers.Remove(Customer);
        }

        public void Update(Customer item)
        {
            _context.Customers.Update(item);
        }

        public bool Save()
        {
            return _context.SaveChanges() >= 0;
        }
    }
}
```

Now we'll create an interface for the repository we just created. Create a file `ICustomerRepository.cs` with the following:

```cs
namespace CSharpRESTDemo.Repositories
{
    public interface ICustomerRepository
    {
        void Add(Customer item);
        void Delete(Guid id);
        IQueryable<Customer> GetAll();
        Customer GetSingle(Guid id);
        bool Save();
        void Update(Customer item);
    }
}
```

The _repository_ pattern in C# mediates between the domain and data mapping layers. It is
mostly used where we need to modify the data before passing to the next stage, providing
a clean separation and one-way dependency between the domain and data mapping layers.

There are _two_ clear advantages:

1. **Improving Testability**. We can create an interface of the repository, and reference it.
We can make a fake object (using `moq` for example) which implements that interface.
2. **Improved Modularity**. We can swap out with various data stores without changing the underlying API.

---

Create a new folder called `Dtos` and create a file `CustomerDto.cs` with the following:

```cs
namespace CSharpRESTDemo.Dtos
{
    public class CustomerDto
    {
        public Guid Id { get; set; }
        public string Firstname { get; set; }
        public string Lastname { get; set; }
        public int Age { get; set; }
    }
}
```

Create a file `CustomerCreateDto.cs` with the following:

```cs
namespace CSharpRESTDemo.Dtos
{
    public class CustomerCreateDto
    {
        public string Firstname { get; set; }
        public string Lastname { get; set; }
        public int Age { get; set; }
    }
}
```

Create a file `CustomerUpdateDto.cs` with the following: (yes, same as previous)

```cs
namespace CSharpRESTDemo.Dtos
{
    public class CustomerUpdateDto
    {
        public string Firstname { get; set; }
        public string Lastname { get; set; }
        public int Age { get; set; }
    }
}
```

A **D**ata **T**ransfer **O**bject (**DTO**) is an object that is used to encapsulate data. As of this moment, our code would map directly to the database fields. That's not always desirable. Sometimes you want to change the shape of the data for the following reasons:

- **Hide** particular properties that clients are not supposed to view.
- **Omit** some properties in order to improve performance.
- **Decouple** the service layer from the database layer.

See [Create Data Transfer Objects](https://docs.microsoft.com/en-us/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5) for more information.

---

In the project folder, create a file `AutoMapping.cs` with the following:

```cs
public class AutoMapping : Profile
{
    public AutoMapping()
    {
        CreateMap<Customer, CustomerDto>();
        CreateMap<CustomerDto, Customer>();

        CreateMap<Customer, CustomerCreateDto>();
        CreateMap<CustomerCreateDto, Customer>();

        CreateMap<Customer, CustomerUpdateDto>();
        CreateMap<CustomerUpdateDto, Customer>();
    }
}
```

This creates a mapping between the DTO and entity objects.

---

Add a folder `Controllers` and create a file `CustomerController.cs` with the following:

```cs
using AutoMapper;
using CSharpRESTDemo.Dtos;
using CSharpRESTDemo.Entities;
using CSharpRESTDemo.Repositories;
using Microsoft.AspNetCore.JsonPatch;
using Microsoft.AspNetCore.Mvc;
using System;
using System.Linq;

namespace CSharpRESTDemo.Controllers
{
    [Route("api/[controller]")]
    public class CustomersController : Controller
    {
        private IMapper _mapper;
        private ICustomerRepository _customerRepository;

        public CustomersController(IMapper mapper, ICustomerRepository customerRepository)
        {
            _mapper = mapper;
            _customerRepository = customerRepository;
        }

        [HttpGet]
        public IActionResult GetAllCustomers()
        {
            var allCustomers = _customerRepository.GetAll().ToList();

            var allCustomersDto = allCustomers.Select(x => _mapper.Map<CustomerDto>(x));

            return Ok(allCustomersDto);
        }

        [HttpGet]
        [Route("{id}", Name = "GetSingleCustomer")]
        public IActionResult GetSingleCustomer(Guid id)
        {
            Customer customerFromRepo = _customerRepository.GetSingle(id);

            if (customerFromRepo == null)
            {
                return NotFound();
            }

            return Ok(_mapper.Map<CustomerDto>(customerFromRepo));
        }

        [HttpPost]
        public IActionResult AddCustomer([FromBody] CustomerCreateDto customerCreateDto)
        {
            Customer toAdd = _mapper.Map<Customer>(customerCreateDto);

            _customerRepository.Add(toAdd);

            bool result = _customerRepository.Save();

            if (!result)
            {
                return new StatusCodeResult(500);
            }

            return CreatedAtRoute("GetSingleCustomer", new { id = toAdd.Id }, _mapper.Map<CustomerDto>(toAdd));
        }

        [HttpPut]
        [Route("{id}")]
        public IActionResult UpdateCustomer(Guid id, [FromBody] CustomerUpdateDto updateDto)
        {
            var existingCustomer = _customerRepository.GetSingle(id);

            if (existingCustomer == null)
            {
                return NotFound();
            }

            _mapper.Map(updateDto, existingCustomer);

            _customerRepository.Update(existingCustomer);

            bool result = _customerRepository.Save();

            if (!result)
            {
                return new StatusCodeResult(500);
            }

            return Ok(_mapper.Map<CustomerDto>(existingCustomer));
        }

        [HttpPatch]
        [Route("{id}")]
        public IActionResult PartiallyUpdate(Guid id, [FromBody] JsonPatchDocument<CustomerUpdateDto> customerPatchDoc)
        {
            if (customerPatchDoc == null)
            {
                return BadRequest();
            }

            var existingCustomer = _customerRepository.GetSingle(id);

            if (existingCustomer == null)
            {
                return NotFound();
            }

            var customerToPatch = _mapper.Map<CustomerUpdateDto>(existingCustomer);
            customerPatchDoc.ApplyTo(customerToPatch);

            _mapper.Map(customerToPatch, existingCustomer);

            _customerRepository.Update(existingCustomer);

            bool result = _customerRepository.Save();

            if (!result)
            {
                return new StatusCodeResult(500);
            }

            return Ok(_mapper.Map<CustomerDto>(existingCustomer));
        }


        [HttpDelete]
        [Route("{id}")]
        public IActionResult Remove(Guid id)
        {
            var existingCustomer = _customerRepository.GetSingle(id);

            if (existingCustomer == null)
            {
                return NotFound();
            }

            _customerRepository.Delete(id);

            bool result = _customerRepository.Save();

            if (!result)
            {
                return new StatusCodeResult(500);
            }

            return NoContent();
        }

    }
}
```

Each method in the `CustomersController` class has an attribute that defines an HTTP verb and route.
When a client interacts with `api/customer` with a `POST` request method, it will execute the code
in the `AddCustomer` method as an example.

Note the fact that we do not alter the shape of the data at this layer. We've been using
_repositories_ and _data transfer objects_ to separate our layers. On a larger scale project,
with many developers involved, having this insulation layer is a life-saver.

---

At this point in time, you should have a working application after an initial migration of
the database.

> The migrations feature in EF Core provides a way to incrementally update the database schema to keep it in sync with the application's data model while preserving existing data in the database.

See [Migrations Overview](https://docs.microsoft.com/en-us/ef/core/managing-schemas/migrations/?tabs=dotnet-core-cli) for more information.

```sh
dotnet ef migrations add InitialCreate
dotnet ef database update
dotnet run
```

Check out [Part Two](@/notes/csharprestapi-2.md)!
