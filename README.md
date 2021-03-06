# Code first with System Versioning Entity Framework Dotnet Core 2x simple guide

## 1. Startup 
This guide it's re-using the sample done in this [other CodeFirst Sample](https://github.com/entityframeworkcore-samples/CodeFirst)

First step is execute the powershell script "1.UpdateDatabase.ps1" which will execute the bellow command in "\DAL\DAL.Jecaestevez.csproj"
> dotnet ef database update --startup-project ..\ConsoleApp

Using Package Manager Console select the DAL.JecaestevezApp.csproj and execute

> PM> update-database �verbose

## 2 Add empty migration 

Create a new empty migration with a name "SystemVersioning"
> dotnet ef migrations add SystemVersioning
> PM> add-migration SystemVersioning

## 3 Modify Migration to change the tables to add  System versioning to existent tables 

This will create a skeleton migration with an Up(MigrationBuilder migrationBuilder) and a Down(MigrationBuilder migrationBuilder) method which contain no code

Modify the skeleton migration as shown below

```
   public partial class SystemVersioning : Migration
    {
		List<string> tablesToUpdate = new List<string>
        {
            "Items"
        };
        protected override void Up(MigrationBuilder migrationBuilder)
        {
            OnMigrationUpSystemVersioning(migrationBuilder);
        }

        protected override void Down(MigrationBuilder migrationBuilder)
        {
            OnMigrationDownSystemVersioning(migrationBuilder);
        }

        private void OnMigrationUpSystemVersioning(MigrationBuilder migrationBuilder)
        {
            migrationBuilder.Sql($"CREATE SCHEMA History");
            foreach (var table in tablesToUpdate)
            {
                string alterStatement = $@"ALTER TABLE {table} ADD SysStartTime datetime2(0) GENERATED ALWAYS AS ROW START HIDDEN
         CONSTRAINT DF_{table}_SysStart DEFAULT GETDATE(), SysEndTime datetime2(0) GENERATED ALWAYS AS ROW END HIDDEN
         CONSTRAINT DF_{table}_SysEnd DEFAULT CONVERT(datetime2 (0), '9999-12-31 23:59:59'),
������   PERIOD FOR SYSTEM_TIME (SysStartTime, SysEndTime)";
                migrationBuilder.Sql(alterStatement);
                alterStatement = $@"ALTER TABLE {table} SET (SYSTEM_VERSIONING = ON (HISTORY_TABLE = History.{table}));";
                migrationBuilder.Sql(alterStatement);
            }
        }
        private void OnMigrationDownSystemVersioning(MigrationBuilder migrationBuilder)
        {
            foreach (var table in tablesToUpdate)
            {
                string alterStatement = $@"ALTER TABLE {table} SET (SYSTEM_VERSIONING = OFF);";
                migrationBuilder.Sql(alterStatement);
                alterStatement = $@"ALTER TABLE {table} DROP PERIOD FOR SYSTEM_TIME";
                migrationBuilder.Sql(alterStatement);
                alterStatement = $@"ALTER TABLE {table} DROP DF_{table}_SysStart, DF_{table}_SysEnd";
                migrationBuilder.Sql(alterStatement);
                alterStatement = $@"ALTER TABLE {table} DROP COLUMN SysStartTime, COLUMN SysEndTime";
                migrationBuilder.Sql(alterStatement);
                alterStatement = $@"DROP TABLE History.{table}";
                migrationBuilder.Sql(alterStatement);
            }
            migrationBuilder.Sql($"DROP SCHEMA History");
        }
    }
```

Apply the migration
> dotnet ef database update
> PM> Update-Database