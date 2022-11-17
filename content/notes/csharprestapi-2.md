+++
title = "REST Web API Introduction in C# - Part Two"
slug = "csharp-rest-api-intro-part-two"
date = 2020-11-23
+++

# Summary

This is the second part of the **REST Web API Introduction in C#** series. If you haven't already,
check out [Part One](@/notes/csharprestapi-1.md)!

# Prerequisites

Please make sure you have [docker](https://docs.docker.com/get-docker/) and [docker-compose](https://docs.docker.com/compose/install/) installed.

# Demonstration Continued

In the **Part One**, we built the majority of our application. It should be functional at this point,
but unrealistic in a production environment. We ideally wouldn't be using an in-memory database, let's change that.

## Switching Databases

In the `Startup` constructor, in the `Startup.cs` file, we load configuration from `appsettings.json`.
We haven't created that yet, so let's do that and add the snippet below. We'll use that to load connection
details to our database.

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=db\\SQLEXPRESS,1433;Database=master;User ID=sa;Password=CHANGEME123!;Integrated Security=False;"
  }
}
```

We're specifying a remote connection to a host known as `db`, which will resolve to an IP address. The reason
for this will become clearer as we complete our `docker` setup. `1433` is the port. The database itself will be an instance of Microsoft SQL Express.
The database name is `master`, with a user `sa` and password `CHANGEME123!`. We also set integrated security to `False` because
we _do_ want to use our specified credentials.

> Integrated Security When false, User ID and Password are specified in the connection. When true, the current Windows account credentials are used for authentication. Recognized values are true, false, yes, no, and sspi (strongly recommended), which is equivalent to true. If User ID and Password are specified and Integrated Security is set to true, the User ID and Password will be ignored and Integrated Security will be used.

See [SqlConnection.ConnectionString](https://docs.microsoft.com/en-us/dotnet/api/system.data.sqlclient.sqlconnection.connectionstring?redirectedfrom=MSDN&view=dotnet-plat-ext-5.0#System_Data_SqlClient_SqlConnection_ConnectionString) for more information.

---

In the same file, under `ConfigureServices`, we'll replace the line:

```cs
services.AddDbContext<DemoDbContext>(opt => opt.UseInMemoryDatabase(databaseName: "InMemoryDb"));
```

With the following:

```cs
string connectionString = Configuration.GetConnectionString("DefaultConnection");
services.AddDbContext<DemoDbContext>(opt => opt.UseSqlServer(connectionString));
```

Now we're pulling the database connection string from the `appsettings.json` file.
In a real environment, you might have different modes of operation (development, staging, production) where you'd want to specify different database connections.

## Docker

In the previous step, we specified a SQL Express database connection. As we add more architectural components and services to our project, it becomes more complicated to deploy or on-board for new developers. [Docker](https://docs.docker.com/) solves this problem.

#### Virtues in a Nutshell

1. Docker enables efficient use of system resources.
2. Docker enables faster software delivery cycles.
3. Docker enables application portability.

Let's start by creating a `Dockerfile` in the parent directory of `CSharpRESTDemo`.

```dockerfile
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
WORKDIR /app

# Copy .csproj and restore as distinct layers.
COPY CSharpRESTDemo/*.csproj ./CSharpRESTDemo/
# Restore dependencies specified in .csproj
RUN dotnet restore CSharpRESTDemo
# Install the dotnet-ef tool.
RUN dotnet tool install --global dotnet-ef

# Copy everything else and build app.
COPY CSharpRESTDemo/. ./CSharpRESTDemo/
COPY ./start.sh ./CSharpRESTDemo/
WORKDIR /app/CSharpRESTDemo
RUN chmod +x ./start.sh

# Execute script when container runs.
CMD /bin/bash ./start.sh
```

- `start.sh` does not exist yet, we'll get to that in the next step.

In this code snippet, we're using a microsoft image with correct dotnet version installed.
We copy over the `CSharpRESTDemo.csproj` file, and install the dependencies specified. We also install the
`dotnet-ef` tool so we can use the **EntityFramework** commands.

Next, we'll copy over the `CSharpRESTDemo` contents into the container.
Then, copying over the `start.sh` and mark it as executable. We'll use this to start up our application.

---

Now we'll create a new file called `start.sh`, it should be in the parent directory of `CSharpRESTDemo` with the `Dockerfile`.
It should have the following contents:

```sh
PATH="$PATH:$HOME/.dotnet/tools/"

set -e

until dotnet ef database update; do
>&2 echo "SQL Server is starting up"
sleep 1
done

>&2 echo "SQL Server is up - executing command"
exec dotnet run
```

This script will attempt to perform database migrations (if needed) and run the application.

---

The next step will allow us to compose our service components. Create a file in the parent drectory of `CSharpRESTDemo` called `docker-compose.yml`, it should have the following contents:

```yaml
version: "3"
services:
  api:
    build: .
    ports:
      - "5000:5000"
      - "5001:5001"
    environment:
      ASPNETCORE_URLS: "http://0.0.0.0:5000;https://0.0.0.0:5001"
  db:
    image: "mcr.microsoft.com/mssql/server:latest"
    environment:
      SA_PASSWORD: "CHANGEME123!"
      ACCEPT_EULA: "Y"
```

In this file, we have _two_ services, `api` and `db`.

Looking at `api`, we tell it to build the `Dockerfile` we created earlier and
make the port range `5000`-`5001` accessible to the host. We also specify an environment variable to tell the application runtime
that we want `kestrel` to bind to the ports we specified.

The `db` service will use a _ready-to-go_ [Microsoft SQL Server](https://hub.docker.com/_/microsoft-mssql-server). We only need to specify `SA_PASSWORD` and `ACCEPT_EULA`. Note that
these are the same credentials we specified in the `appsettings.json` file earlier.

---

Now you should be able to run `docker-compose up` and access the API on `http://localhost:5000`!

Check out the **Part Three** for more!
