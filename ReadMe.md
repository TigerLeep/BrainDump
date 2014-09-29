#Lee's Brain Dump

[Copying binary files to SQL Server without mapped drive](CopyBinaryFilesToSqlServerWithoutMappedDrive.md)

* Read binary file as bytes and store in SQL Server table in varbinary field
* Tell SQL Server to write binary from varbinary to harddrive
	* Use xp_cmdshell to execute bcp
	* Use xp_UA* to create and use ADODB.Stream object
*    