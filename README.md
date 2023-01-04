# Assignment02

# Set the parameters for the resource group, Azure SQL database, and web app
$resourceGroupName = "rg-CMPE363-assignment2-TARIKBERKtest3"
$location = "eastus"
$sqlServerName = "tarikberksql1"
$databaseName = "tarikberkdb1"
$webAppNamePrefix = "webapp-CMPE363-assignment2-TARIKBERKtest3-"

# Create the resource group
New-AzResourceGroup -Name $resourceGroupName -Location $location

# Create an Azure SQL Server
$sqlServer = New-AzSqlServer -ResourceGroupName $resourceGroupName `
  -ServerName $sqlServerName `
  -Location $location `
  -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential -ArgumentList "username", $(ConvertTo-SecureString -String "P@ssw0rd123" -AsPlainText -Force))

# Create an Azure SQL database
$database = New-AzSqlDatabase -ResourceGroupName $resourceGroupName `
  -ServerName $sqlServerName `
  -DatabaseName $databaseName

Get-AzSqlServer -ResourceGroupName $resourceGroupName -ServerName $sqlServerName
Get-AzSqlDatabase -ResourceGroupName $resourceGroupName -ServerName $sqlServerName -DatabaseName $databaseName

# Connect to the Azure SQL database
# Use the correct username and password in the connection string
$connectionString = "Server=tcp:$sqlServerName.database.windows.net,1433;Initial Catalog=$databaseName;Persist Security Info=False;User ID=username;Password=P@ssw0rd123;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;"
$connection = New-Object System.Data.SqlClient.SqlConnection($connectionString)
$connection.Open()

# Create the table
$command = $connection.CreateCommand()
$command.CommandText = "CREATE TABLE tblEmployee (EmpName varchar(50), EmpSurname varchar(50), EmpAddress varchar(50), EmpPhone varchar(50))"
$command.ExecuteNonQuery()

# Insert random data into the table
$random = New-Object System.Random
for ($i = 0; $i -lt 50; $i++) {
  $command.CommandText = "INSERT INTO tblEmployee (EmpName, EmpSurname, EmpAddress, EmpPhone) VALUES ('Name $i', 'Surname $i', 'Address $i', 'Phone $i')"
  $command.ExecuteNonQuery()
}

# Close the connection
$connection.Close()

# Create the web app
for ($i = 0; $i -lt $WebAppCount; $i++) {
  $webAppName = "$webAppNamePrefix$i"
  New-AzWebApp -Name $webAppName -ResourceGroupName $resourceGroupName -Location $location
}
