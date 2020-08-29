# Connecting to Azure resources securly using Managed Identities

Accoding to Microsoft, "A common challenge when building cloud applications is how to manage the credentials in your code for authenticating to cloud services. Keeping the credentials secure is an important task. Ideally, the credentials never appear on developer workstations and aren't checked into source control. Azure Key Vault provides a way to securely store credentials, secrets, and other keys, but your code has to authenticate to Key Vault to retrieve them.

The managed identities for Azure resources feature in Azure Active Directory (Azure AD) solves this problem. The feature provides Azure services with an automatically managed identity in Azure AD. You can use the identity to authenticate to any service that supports Azure AD authentication, including Key Vault, without any credentials in your code."
https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview

Saving credentials inside codes is a definite NO NO, there goes SQL authentication out of the window. Creating managed identiy is no big deal but what if you already have Service principal created for the same purpose. Why you would choose Managed Identity over Service Principal? Internally, managed identities are service principals of a special type, which are locked to only be used with Azure resources. When the managed identity is deleted, the corresponding service principal is automatically removed. But what is the advantage? If a system-assigned Managed identity is removed, it also removes the Service Principal created behind the scenes.


Use case;
Create a Secure Link between Azure Data Factory and Azure SQL Database to copy data from one table to another. Before copying data, first, data in the destination table should be dropped.

Note that Managed identity makes more sense when a custom built web application is needed to connect to Azure Resources.

Steps;
1. Add an Active Directory Admin to SQL Server. This is needed because SQL Server doesn't let non Active Directory users add Active Directory accounts. So, SQL authentication won't let anyone create an Active Directory user. As discussed before, the Managed Principal is an Active Directory account. Therefore before adding Managed Principal, first, SQL should grant access to Azure Active Directory users.
2. Create Azure Key vault. To avoid anyone seeing SQL Server credentials, it's best to keep such information in Azure Key Vault. This is very much applicable when the code is saved in GitHub. Key vault should store database connection link as secret; Server=tcp:<<<SQL Server Name>>>.database.windows.net,1433;Database=<SQL Database Name>
3. In Azure Data Factory Linked Service, select Azure Key Vault and select Managed Identity as Authentication Type. Azure should show the system-assigned managed identity just below the Authentication Type combo box, and the Managed Identity name is the same as Azure Data Factory name.
4. In this step, Managed Identity is granted SQL database privileges. Connect to SSMS using Azure Active Directory Authentication. First, create a user with the same name as Managed Identity. Then give minimum database privileges. Usually, db_datareader and db_datawriter is sufficient, but because of the truncation command, db_ddladmin is also granted.

CREATE USER <Managed Identity Name> FROM EXTERNAL PROVIDER

ALTER ROLE db_datareader ADD MEMBER <Managed Identity Name>
ALTER ROLE db_datawriter ADD MEMBER <Managed Identity Name>
ALTER ROLE db_ddladmin ADD MEMBER <Managed Identity Name>

