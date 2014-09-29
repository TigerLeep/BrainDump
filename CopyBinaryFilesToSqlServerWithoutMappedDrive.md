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
