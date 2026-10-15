# syntax=docker/dockerfile:1

# The build context is the repository root:
#   docker build -f src/Web/Dockerfile -t cloudshirt/web .

# build
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src

# Copy only the project files before restoring, so editing application code
# reuses this layer instead of downloading every package again.
COPY Directory.Packages.props global.json ./
COPY src/ApplicationCore/ApplicationCore.csproj src/ApplicationCore/
COPY src/BlazorAdmin/BlazorAdmin.csproj src/BlazorAdmin/
COPY src/BlazorShared/BlazorShared.csproj src/BlazorShared/
COPY src/Infrastructure/Infrastructure.csproj src/Infrastructure/
COPY src/Web/Web.csproj src/Web/
RUN dotnet restore src/Web/Web.csproj

COPY . .

RUN dotnet publish src/Web/Web.csproj -c Release -o /app/publish

# --- runtime ----------------------------------------------------------------
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish ./

# Program.cs only reads the connection strings from configuration when the
# environment is Development or Docker. Any other value sends startup to Azure
# Key Vault, which this deployment does not have, and the application fails
# before it listens. The connection strings themselves are injected at run
# time and are not part of the image.
ENV ASPNETCORE_ENVIRONMENT=Development \
    ASPNETCORE_URLS=http://+:80 \
    DOTNET_EnableDiagnostics=0

EXPOSE 80

ENTRYPOINT ["dotnet", "Web.dll"]
