
## Azure and SQL Server access ##

Azure and SQL Server access

Azure SQL Databases support Contained Users - a feature available on SQL Server since v2012. 

### From the article "Azure SQL Database access control"
>User accounts can be created in the master database and can be granted permissions in all databases on the server, or they can be created in the database itself (called contained users). For information on creating and managing logins, see Manage logins. To enhance portability and scalabilty, use contained database users to enhance scalability. For more information on contained users, see Contained Database Users - Making Your Database Portable, CREATE USER (Transact-SQL), and Contained Databases.+
>As a best practice your application should use a dedicated account to authenticate -- this way you can limit the permissions granted to the application and reduce the risks of malicious activity in case your application code is vulnerable to a SQL injection attack. The recommended approach is to create a contained database user, which allows your app to authenticate directly to the database.

[Azure SQL Database access control](https://docs.microsoft.com/en-us/azure/sql-database/sql-database-control-access)
[Contained Database Users - Making Your Database Portable](https://docs.microsoft.com/en-us/sql/relational-databases/security/contained-database-users-making-your-database-portable)
[Contained Databases] (https://docs.microsoft.com/en-us/sql/relational-databases/databases/contained-databases)

### Creating a Contained User
```sql
USE [MyDatabase]
GO

DROP USER [MyContainedUser]
GO

CREATE USER [MyContainedUser] WITH PASSWORD='**PASSWORD**', DEFAULT_SCHEMA=[dbo]
GO

ALTER ROLE db_datareader ADD MEMBER [MyContainedUser];
ALTER ROLE db_datawriter ADD MEMBER [MyContainedUser];
GRANT CONNECT TO [MyContainedUser];
GO
```