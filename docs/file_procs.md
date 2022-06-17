# QSF File-related Sub Routines

## ParseFileURI
###	Purpose
This function parses a string containing a fully-qualified file name and returns the elements.

###	Inputs
	\_sInput: The string to be parsed
###	Output
	sUpperPath: Everything to the left of the last slash.  Begins without a slash; ends with a slash.
	sFileName:  Everything between the last slash and the last period
	sExt: 		Everything to the right of the last period


### Example:

	ParseFileURI( '\some\upper/path/some_file.txt', var1, var2, var3) will return:
	
	sInput 	= '/some/upper/path/some_file.txt'
	var1	= 'some/upper/path/'
	var2	= 'some_file'
	var3	= 'txt'



##   SetFileURI
### Prupose
This function constructs a Fully-Qualified File Name (FQFN) from the provided elements.

### Inputs
		\_sSourceLib: The data connection name (the source library)
		\_sUpperPath: A path occuring between the source library and the file name 
		\_sSourceTable: The file name
		\_sExt: The file extension

### Output
		sResultURI: The fully-qualified path to a file 


### Example:

	SetFileURI( 'Scripts', '\some\upper/path/', 'some_file', '.qvs') 
	
will return:
	
	sResultURI 	= 'Scripts/some/upper/path/some_file.qvs'

## FileExists
### Prupose
Return a boolean indicating whether a file exists
### Inputs
- \_sFileURI: a fully-qualified URI to the file.  (ex. lib://Scripts/some/upper/path/some_file.txt)
### Outputs
- bExists: a boolean indicating whether the file exists.

## GetFileDetails
### Prupose
Get some information about a file including whether it exists, the size, and the last modified time.
### Inputs
- \_sFileURI: a fully-qualified URI to the file.  (ex. lib://Scripts/some/upper/path/some_file.txt)
### Outputs
- bIsPresent: a boolean indicating whether the file exists.
- iFileSize: the size of the file in bytes.  an integer.
- tFileTime: a timestamp representing the last modified time of the file.

