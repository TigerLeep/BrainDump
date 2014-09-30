#Copy Binary Files To SQL Server Without Mapped Drive

* Read binary file as bytes and store in SQL Server table in varbinary field
* Tell SQL Server to write binary from varbinary to harddrive
	* Use xp_cmdshell to execute bcp<br />
	  or
    * Use xp_UA* to create and use ADODB.Stream object

Read binary File...
-------------------
```c#
string connectionString = "...";
string filename = @"C:\Projects\Installer ASP\IssueTrakInstaller\ProgramFiles\IssueTrakLibrary_32.dll";
byte[] fileBytes = File.ReadAllBytes(filename);
using(var connection = new SqlConnection(connectionString))
{
  using (var command = new SqlCommand())
  {
    connection.Open();
    command.Connection = connection;
    command.CommandText = "INSERT INTO Installer_Temp (Filename, Data) VALUES (@Filename, @Data)";
    command.Parameters.Add("@Filename", SqlDbType.NVarChar, 255).Value = "IssueTrakLibrary_32.dll";
    command.Parameters.Add("@Data", SqlDbType.VarBinary, fileBytes.Length).Value = fileBytes;
    command.CommandTimeout = 60;
    command.ExecuteNonQuery();
  }
}
```
Write Binary File (bcp)
-----------------------
```sql
CREATE PROCEDURE [dbo].[ExportBinaryFileForInstaller]
(
  @filenameWithoutPath nvarchar(255),
  @destinationPathEndingWithSlash nvarchar(255),
  @adminUserName nvarchar(255),
  @adminPassword nvarchar(255),
  @databaseName nvarchar(255)
)
AS
BEGIN
  DECLARE @formatFile nvarchar(255), @binaryFile nvarchar(255)

  SELECT @formatFile = @destinationPathEndingWithSlash + 'exportbinaryfile.fmt'
  SELECT @binaryFile = @destinationPathEndingWithSlash + @filenameWithoutPath

  DECLARE @command nvarchar(4000)

  SELECT @command = 'echo 10.0 > "'+ @formatFile + '"'
  EXEC xp_cmdshell @command

  SELECT @command = 'echo 1 >> "'+ @formatFile + '"'
  EXEC xp_cmdshell @command

  SELECT @command = 'echo 1 SQLBINARY 0 0 "" 1 Data "" >> "'+ @formatFile + '"'
  EXEC xp_cmdshell @command

  SELECT @command = 'bcp "SELECT Data FROM Installer_Temp WHERE Filename = ''' + @filenameWithoutPath + '''"'
                    + ' queryout "' + @binaryFile + '"'
                    + ' -S localhost'
                    + ' -U ' + @adminUserName
                    + ' -P ' + @adminPassword
                    + ' -d ' + @databaseName
                    + ' -f ' + @formatFile
  EXEC xp_cmdshell @command

  SELECT @command = 'del ' + @formatFile
  EXEC xp_cmdshell @command
END
```
