# QSF General Sub Routines
## GetEnvFromAppName
### Purpose
This function searches the provided string for the following reserved words (DEV|TEST|UAT|PROD) and sets the global 
variable sReturnEnv accordingly.  If one of the above words is found, sReturnEnv will be set to the first word found, 
searching in the following order:
  1. PROD
  2. UAT
  3. TEST
  4. DEV

### Inputs
		**\_sAppNameOrig**: String.  The name of the application.
		**\_sDefaultEnv**: String.  The default environment.  

### Outputs
		**sReturnEnv**: String.  The resulting environment.  Default is _sDefaultEnv.


### Examples
If none of the above reserved words are found, `sReturnEnv` will be set to _sDefaultEnv.

Example:
	GetEnvFromAppName( 'This     is my test app prod', 'DEV')  -->   'PROD'
	GetEnvFromAppName( 'This     is my test app ', 'DEV')  	-->    'TEST'
	GetEnvFromAppName( 'This  !%$^[]  is my app ', 'DEV')  		-->   'DEV' 


## CleanseString
### Purpose
This function parses a string (_sInput) and replaces repeating occurrences of _sSearch with _sReplace.

### Inputs
		**\_sInput**: The string to manipulated
		**\_sSearch**: The string to be searched for
		**\_sReplace**: The string to replace with

### Output
		**sOutput**: String - the cleansed string


### Example:

	CleanseString( 'This     is an     example', ' ', '_') = 'This_is_an_example'

## shortenAppName
### Purpose
A wrapper for the calling the CleanseString function specifically to replace spaces with underscores.

### Inputs

		_sInput: String.  The string to be cleansed.
		
### Outputs

		sOutput: The cleansed string.  In this case a string where spaces have been replaced with underscores.

### Example: 

	Call shortenAppName('This     is my test app prod') -->  'This_is_my_test_app_prod'
