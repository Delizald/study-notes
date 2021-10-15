**Install Entity framework**

  dotnet add package Microsoft.EntityFrameworkCore.Sqlite

or

  dotnet add package ${provider}

Current providers: https://docs.microsoft.com/en-us/ef/core/providers/?tabs=dotnet-core-cli

**run app**

dotnet  run

**Install entity framework globally**
dotnet tool install --global dotnet-ef

dotnet ef database update