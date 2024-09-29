# Use the official .NET 7 SDK as a build environment
FROM mcr.microsoft.com/dotnet/aspnet:7.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

# Use the official .NET SDK for building the application
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /src
COPY ["Bloggie.Web.csproj", "./"]
RUN dotnet restore "./Bloggie.Web.csproj"  # Restores NuGet packages

# Copies the entire project and builds it
COPY . .
WORKDIR "/src/"
RUN dotnet build "Bloggie.Web.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "Bloggie.Web.csproj" -c Release -o /app/publish

# Use the base image to create a final Docker image for production
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Bloggie.Web.dll"]
