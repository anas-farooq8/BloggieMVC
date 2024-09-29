# Dockerized ASP.NET Web Application with PostgreSQL and pgAdmin

This project demonstrates how to containerize an ASP.NET Core web application along with a PostgreSQL database and pgAdmin for database management. Docker and Docker Compose are used to manage the containers and handle communication between them.

## Project Structure

```
/BloggieMVC
│
├── Dockerfile                      # Dockerfile for the ASP.NET web application
├── docker-compose.yml              # Docker Compose file to define services
├── Bloggie.Web.csproj              # Your ASP.NET Core project file
└── ...                             # Other project files
```

## Prerequisites

Before getting started, we need following installed on our machine:

- Docker
- Docker Compose

## Steps to Set Up the Environment

### 1. **Create the Dockerfile for the ASP.NET Web Application**

Create a file named `Dockerfile` in the root of your project directory with the following content:

```dockerfile
# Use the official .NET 7 SDK as a build environment
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

# Use the official .NET SDK for building the application
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["Bloggie.Web.csproj", "./"]  # Replace "Bloggie.Web" with the name of your project
RUN dotnet restore "./Bloggie.Web.csproj"  # Restores NuGet packages

COPY . .  # Copies the entire project
WORKDIR "/src/"
RUN dotnet build "Bloggie.Web.csproj" -c Release -o /app/build  # Builds the project

FROM build AS publish
RUN dotnet publish "Bloggie.Web.csproj" -c Release -o /app/publish  # Publishes the app

# Use the base image to create a final Docker image for production
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Bloggie.Web.dll"]  # Replace "YourApp.dll" with your app's DLL file
```

### 2. **Create the Docker Compose File**

Create a file named `docker-compose.yml` in the root of your project directory with the following content:

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:latest
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - bloggie_network

  pgadmin:
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "8080:80"
    depends_on:
      - postgres
    networks:
      - bloggie_network

  webapp:
    build:
      context: .
      dockerfile: Dockerfile # Ensure your Dockerfile is named correctly
    ports:
      - "8000:80"
    depends_on:
      - postgres
    environment:
      - ASPNETCORE_ENVIRONMENT=Development
      - ConnectionStrings__BloggieDbConnectionString=Host=postgres;Port=5432;Database=BloggieDb;Username=postgres;Password=postgres
      - ConnectionStrings__BloggieAuthDbConnectionString=Host=postgres;Port=5432;Database=BloggieAuthDb;Username=postgres;Password=postgres
    networks:
      - bloggie_network

volumes:
  postgres_data:

networks:
  bloggie_network:
    driver: bridge
```

### 3. **Build and Run the Containers**

To build and run your Docker containers, navigate to the root directory of your project in the terminal and execute the following commands:

1. **Build the Docker images**:

   ```bash
   docker-compose build
   ```

2. **Run the Docker containers**:
   ```bash
   docker-compose up -d
   ```

This will start three containers: PostgreSQL, pgAdmin, and your ASP.NET web application.

### 4. **Access the Services**

- **Web Application**: Your ASP.NET web application will be available at `http://localhost:8000`.
- **pgAdmin**: Access pgAdmin at `http://localhost:8080`, login with `admin@admin.com` and password `admin`. You can use pgAdmin to manage your PostgreSQL databases.

### 5. **Handling Database Migrations**

To apply migrations from within the web app container, execute the following command:

1. Access the web app container:

   ```bash
   docker exec -it <webapp-container-id> bash
   ```

2. Inside the container, run:
   ```bash
   dotnet ef database update -Context AuthDbContext
   ```

This will apply any pending migrations to your PostgreSQL database.

## Explanation of the Setup

### Containers

- **PostgreSQL**: The database server where your application data is stored. We set environment variables to define the default username and password, and we use a Docker volume to persist the data even if the container stops.
- **pgAdmin**: A web-based database management tool for PostgreSQL. It runs in its own container and connects to the PostgreSQL container to manage the databases.
- **ASP.NET Web Application**: Your web app container that runs on .NET 7. It connects to the PostgreSQL container using the defined connection strings.

### Communication Between Containers

The services are defined within the same Docker Compose file and are connected to the same Docker network (`bloggie_network`). This allows the web application to reference the PostgreSQL container by its service name (`postgres`) in the connection strings.

### Volumes

- **Volumes** in Docker allow you to persist data generated by and used by Docker containers. In this setup, the PostgreSQL container uses a volume (`postgres_data`) to store database data, ensuring that it is not lost when the container stops or is recreated.

## Conclusion

This guide walks you through the process of setting up a Docker environment with PostgreSQL, pgAdmin, and an ASP.NET web application. By following these steps, you can easily manage your database and application in isolated containers, ensuring a clean and efficient development process.

For further customization or scaling, feel free to modify the Docker Compose file and Dockerfile as needed.
