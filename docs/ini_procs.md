# QSF Configuration Manager
Sub procedures for reading in configuration files and setting parameters.

## The INI file format
The INI file format is an informal standard for configuration files for some computing platforms or software. INI files are simple text files with a basic structure composed of sections, properties, and values.
The name "INI file" comes from the commonly used filename extension .INI, which stands for "initialization".

### Keys (properties)
The basic element contained in an INI file is the key or property. Every key has a name and a value, delimited by an equals sign (=). The name appears to the left of the equals sign.

key=value

### Sections
Keys may (but need not) be grouped into arbitrarily named sections. The section name appears on a line by itself, in square brackets ([ and ]). All keys after the section declaration are associated with that section. There is no explicit "end of section" delimiter; sections end at the next section declaration, or the end of the file. Sections may not be nested.

[section]
a=a
b=b
;c=c    		(commented out with a semi-colon)
#c=c    		(commented out with a hash symbol)
d=a,b,c,d		(can be a comma-separated list of values)

### Additional Reading
You can read about INI files here:
https://en.wikipedia.org/wiki/INI_file


## Usage
-----
1. Call InitParamsTable()
2. Call ReadIniFile( <configFile>)
3. (optional) Call SetParameters( <yourConfigFile>)


### InitParamsTable
InitParamsTable() creates an empty table where the rows from the config file will be loaded.  No parameters are passed to the procedure.  This is done so that subsequent LOAD statements will append the table without a CONCATENATE statement.

### ReadIniFile
The ReadIniFile( <configFile>) procedure will load the [raw_config] table with the rows from the <configFile>.
<configFile> is a fully-qualified path to your file, starting with "lib://" and including the extension (if there is one).

You can pass any filename to the ReadIniFile() procedure.
This means you could use an extension other than .ini. 
You could use .cfg, .conf, .config, or .txt.

For example: "lib://myDataConnection/config.ini"

### SetParameters
Once loaded, the SetParameters() function can be called to set variables (in Qlik) for each key-value pair.
If a key is in a section, the variable will be named section.key, otherwise it will just be key.


## Initiate the Params table.
Initialize the [raw_config] and [config] tables.  [raw_config] is used to see exactly what was loaded from file, including blank rows.  Each row is tagged with the file name.

### Table: raw_config 
RawFileName 	- the fully-qualified path of the <configFile> that was read in.  Includes the "lib://" prefix and extension.
RawContent 		- the content that was read in from each row.  File is read in as a single field (no delimiters).
IsSectionHeader	- [True | False] depending on the presence of the '[' character.
RawSection		- If [IsSectionHeader]=True then this field will contain the section name (between the square brackets).


The [config] table will have the final configurations.

### Table: config
File 			- Same as [raw_config].[RawFileName]
Section			- The section name.  Present (in every row) for key-value pairs occurring in a section.
Content 		- Trimmed [RawContent] (from the file).
ContainsEquals	- [True | False] depending on the presence of an '=' sign.
ContainsComma	- [True | False] depending on the presence of a comma (',').
ContentType 	- [Parameter | List | Unknown].  Note: Only parameters and lists make it to the [config] table.
Key				- If [ContainsEquals]=True, anything to the left of the equals sign.
Value			- If [ContainsEquals]=True, anything to the right of the equals sign.
FirstChar		- ASCII value of the first character of [Value].
IsQuoted		- [True | False] depending on the presence of a single or double quote character in the first position.

Note: [raw_config].[RawSection] is only filled in for the header records, whereas each row in the [config] table will have [Section] (if applicable).
