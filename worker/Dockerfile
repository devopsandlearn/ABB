# Build Stage
FROM mcr.microsoft.com/dotnet/sdk:7.0 AS build
WORKDIR /app

# Copy project files and restore dependencies
COPY *.csproj .
RUN dotnet restore

# Copy all files and build the application
COPY . .
RUN dotnet publish -c Release -o /app/publish

# Runtime Stage
FROM mcr.microsoft.com/dotnet/runtime:7.0
WORKDIR /app

# Copy the build output from the build stage
COPY --from=build /app/publish .

# Run the application
ENTRYPOINT ["dotnet", "Worker.dll"]
