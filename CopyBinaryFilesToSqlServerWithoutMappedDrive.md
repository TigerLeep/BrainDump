# Copy Binary Files To SQL Server Without Mapped Drive

* Read binary file as bytes and store in SQL Server table in varbinary field
* Tell SQL Server to write binary from varbinary to a file
	* Use xp_cmdshell to execute bcp<br />
	  or
    * Use xp_UA* to create and use ADODB.Stream object

## Read binary File...

To send the binary file to SQL Server, we first read the bytes of the file into byte array.  Then we use ADO.NET to INSERT them into a table with a varbinary column along with a unique key (in the example below the unique key is the filename).
```sql
CREATE TABLE [dbo].[BinaryTransfer](
	[Filename] [nvarchar](255) NOT NULL,
	[Data] [varbinary](MAX) NOT NULL
)

```
```c#
string connectionString = "...";
string filename = @"C:\PathToMyFile\MyBinaryFile.dll";
byte[] fileBytes = File.ReadAllBytes(filename);
using(var connection = new SqlConnection(connectionString))
{
  using (var command = new SqlCommand())
  {
    connection.Open();
    command.Connection = connection;
    command.CommandText = "INSERT INTO BinaryTransfer (Filename, Data) VALUES (@Filename, @Data)";
    command.Parameters.Add("@Filename", SqlDbType.NVarChar, 255).Value = "MyBinaryFile.dll";
    command.Parameters.Add("@Data", SqlDbType.VarBinary, fileBytes.Length).Value = fileBytes;
    command.CommandTimeout = 60;
    command.ExecuteNonQuery();
  }
}
```

## Tell SQL Server to write binary file...

Once SQL Server has the contents of the file in a varbinary column, we can tell SQL Server to read that varbinary and write it out to a file.  We have a couple of options available to achieve this. The first is using the command line utility bcp.exe and the second is to use the ADO object "ADODB.Stream" to write the bytes out to a file.

#fiehwoihefejfpowjef