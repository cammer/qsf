# Syntax Conventions

A note regarding variable naming convention....

Variable names prefixed with an underscore are sub routine parameters.
Variables appearing without an underscore in a sub routine declaration are return values.

The first character of the variable name indicates the intended data type.  Note that these variables are not actually typed.

Capitalized variable names are global variables.  These do not have types.

## Types
- s = string
- i = integer
- b = boolean

For example:

	Sub ManipulateString( _sInput, sOutput)

		\_sInput 	- is an input parameter of type string
		sOutput 	- is the output parameter of type string

	\iStartNum	- a variable of type int 
	sSomeValue	- a variable of type string
	SERVER.ENV 	- a global variable (untyped)

