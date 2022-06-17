# Qlik Scripting Framework (QSF)

# Purpose
QSF is a qlik scripting library inteneded to be used by all your Qlik Sense applications.  The scripting library is invoked at the beginning of each application's data load.  The library creates a framework which enables the developer/administrator to do the following:
- Establish virtual environments on a single Qlik Sense server
- Easily swap load scripts between applications
- Easily copy applications and load scripts between virtual environments
- Change application load behaviors by simply changing an `.ini` file

# Background
All Qlik Sense applications contain a data load script.  The only way to edit the script is through the web IDE.  We can export the application -- as a `.qvf` file -- and checked it into git, but it's a binary.  So, no `diff` commands.

Thankfully, the Qlik language has an `include` statement.  This means we can save our scripts to disk and, because they are ASCII files, we can use our favourite editor *and* perform diffs.  Yay!

We now have the opportunity to reuse code too!  With multiple projects checked-out to the same root directory, it's easy to call procedures from other libraries.
This is where the *Qlik Scripting Framework* comes in.  *QSF* is a shared library that, when called first, establishes a framework for Qlik Sense applications and their scripts.

# Installation
1. Create the following directory on your server:  `C:\Qlik Share\Scripts`.  This is where your load scripts will reside.  Assuming your scripts are spread across multiple git repositories, you will clone them beneath this root.
1. Clone the QSF repo into this directory. (`C:\Qlik Share\Scripts\qsf`)
2. Copy the `config.ini` file from the `config` folder to the root (`QSF/config/config.ini --> QSF/config.ini`)
3. Edit the `config.ini`

**Note**: The `QSF/config/config.ini` file is a template.  At runtime, *QSF* reads the `config.ini` file from the root folder (`QSF/config.ini`).  It does **not** read the file from the `QSF/config` folder.  This may be confusing because there is no `QSF/config.ini` file checked into the repo.  This was intentional.  We did not want instance-specific values being written back to the repo.

## Server Environment
The `config.ini` describes the server environment and sets the log level.

```
[SERVER]
ENV=DEV
DEBUG=0b0
```

| Variable | Description |
|--|--|
|`DOMAIN_NAME`| The Qlik server's domain name. |
|`ENV`| The environment of the server.  Values = `DEV\|TEST\|UAT\|PROD` |
|`DEBUG`| The logs are more verbose when set to `TRUE`.  Default = FALSE. |

The server environment denotes the default environment for this server.  More on this later.

# How to use QSF in your applications?
1. Change your application's data load script as follows:

```
APP.REPO_NAME='some_repo';
$(include="lib://Scripts/qsf/main.qvs");
```
That's it!

The second line of code is the same across _all_ applications.
The first line of code tells QSF where to run your load scripts from.  QSF is expecting the `APP.REPO_NAME` variable to be defined and non-empty.  The `APP.REPO_NAME` should correspond to a git repo that you have cloned to the `Scripts` directory on the server.

# How it works
The application will load the QSF script first `lib://Scripts/qsf/main.qvs` and *then* call your load script.  By doing this, we establish some universal ground rules for our Qlik Sense applications.

## Application Name
QSF parses the calling application's name and does two things:
1. It looks for the following reserved words which denote an environment: `DEV|TEST|UAT|PROD`
2. It shortens the application name (in a predictable way):

The name shortening algorithm does the following:
1. Removes the above reserved words
2. Removes all special characters 
4. Replaces spaces and dashes with underscores
5. Eliminates repeating underscores

This yields the `APP.SHORT_NAME`.
Here are some examples:
| Application Name | Environment | Short App Name |
|------|--------|-------|
| MY cool \*app name? 21  | N/A | my_cool_app_name_21 | 
| What the\_dev is this? | DEV | what_the_is_this |
| My\_app\_UAT | UAT | my_app |
| My app [uat] | UAT | my_app |
| [Test] My app | TEST | my_app |

## Configuration File
Next, QSF is expecting a configuration file of the following name: `<APP.SHORT_NAME>.ini` in the root of your script repo.  (i.e. `some_repo/<APP.SHORT_NAME>.ini`).

- **Note**: a repo can contain several configuration files, each corresponding to a different Qlik Sense application.
- **Note**: The git repository name does *not* have to match the Qlik application name and vice versa.

This configuration file describes your application.  Here's what the file typically looks like:
```
app_var1=value1
app_var2=value2
# Commented row

[APP]
CLEAR=0b0
DEBUG=0b0
LIMIT_ROWS=0b0
ROW_LIMIT=1000
SCRIPT_NAME=some_other_script
```

**Note**: lines can be commented out using ';' or '#'.
**Note**: 0b prefix is used for binary values (similar to the 0x prefix for Hex).
```
0b0 = FALSE
0b1 = TRUE
```

### Config Variables
| Variable | Description |
|--|--|
|`CLEAR` | Whether the data should be cleared after building the app.  This results in a smaller file size when exporting the app.  Default = FALSE.
|`DEBUG` | The logs are more verbose when set to `TRUE`.  Default = FALSE.
|`DISABLE_DSR`| Disables row-level security (called "Data Set Reduction" in Qlik).  Default = FALSE.
|`LIMIT_ROWS`| Whether or not to limit the number of rows that are loaded to the application.  Effective when testing the application build.  Default = FALSE.
|`ROW_LIMIT`| If `LIMIT_ROWS = TRUE` then only this number of rows will be loaded.
|`SCRIPT_NAME`| The name of the script that will be run when QSF is finished.  Will be suffixed with `.qvs`  Default = `SHORT_APP_NAME`.

### Variables
Other script variables can be defined outside/above of the `[APP]` block.  These variables will be available to your load script as if they had been declared in your load script using the `SET|LET` commands.

## Your Load Script
Once *QSF* has read your application's config file, it will run your load script.  
*QSF* will look for the following file: `Scripts/<APP.REPO_NAME>/<SCRIPT_NAME>.qvs`.  
If you haven't specified a value for the `SCRIPT_NAME` variable, then *QSF* will look for `Scripts/<APP.REPO_NAME>/<APP.SHORT_NAME>.qvs'.

**Note**: your main script can reference *other* scripts using the `include` command.  

## Virtual Environments
To this point, we've considered load script repos that are cloned to a single `Scripts` directory on the server.  *QSF* provides a framework for defining virtual environments.  This means that we can have environment sub-directories under the `Scripts` directory.

Your application can be named anything, but if it contains an environment reserved word in the title, the load scripts will be read from a different sub-directory.  This allows us to have different versions of the same repository cloned on the server.  

### Example
Let's assume that we have the following server configuration.
```
[SERVER]
ENV=DEV
DEBUG=0b0
```

Further, let's assume that we've got `some_repo` cloned across DEV, TEST, and UAT virtual environments.  Then, we would have the following directory structure on the server.

```
C:\Qlik Share\Scripts\some_repo    <-- DEV
C:\Qlik Share\Scripts\_TEST\some_repo
C:\Qlik Share\Scripts\_UAT\some_repo
```

**Note**: You will need to create the environment sub-directories on the server manually.
**Note**: the virtual environment names are prefixed with an underscore.

Notice that the `DEV` environment is not distinguished by a sub-directory because it is the default server environment.

Here are some examples.  In all instances the `APP.REPO_NAME=some_repo` and the main script is `main.qvs`.

### Virtual Environments and Data
Frequently, with Qlik Sense apps, we will read and write data from files.  Qlik's proprietary storage format is the `.QVD` file.  The above virtual environments are also used with data directories.  For example, the server might have the `E:` drive dedicated to data.  This structure would provide a mechanism for virtual environments on the data side:

```
E:\Landing    <-- DEV
E:\_TEST\Landing
E:\_UAT\Landing
```

With this setup, we can have a separate "Landing" area in our DEV, TEST, and UAT (virtual) environments.

# Referencing QSF Environment Variables in Your Script
The following variables are available for referencing in your script.

## Server-Level Global Variables
- SERVER.SCRIPT_LIB       = `Scripts`  
- SERVER.QSF_REPO_NAME    = `qsf`
- SERVER.QSF_LIB          = `Scripts/qsf`
- SERVER.CONFIG_FILE      = `Scripts/qsf/config.ini`
- SERVER.ENV              = As read from the server config file

## Application-Level Global Variables
- APP.NAME        = `DocumentTitle()`
- APP.SHORT_NAME  = Parsed <APP.NAME>
- APP.REPO_NAME   = Whatever was passed from the application.  The application's script directory under the `Scripts` root.
- APP.SCRIPT_LIB  = <SERVER.SCRIPT_LIB>/<APP.REPO_NAME>
- APP.CONFIG_FILE = <APP.SCRIPT_LIB>/<APP.SHORT_NAME>.ini
- APP.ENV         = As read from config file
- APP.CLEAR       = As read from config file
- APP.DEBUG       = As read from config file
- APP.LIMIT_ROWS  = As read from config file
- APP.ROW_LIMIT   = As read from config file
- APP.SCRIPT_NAME = As read from config file
- APP.SCRIPT_FILE = <APP.SCRIPT_LIB>/<APP.SCRIPT_NAME>.qvs

- LOAD_PREFIX_FIRST = `FIRST <APP.ROW_LIMIT>`  Can be used in a `LOAD` statement to limit rows.

Variables ending in `_LIB` are intended to be used with the `lib://` prefix in `LOAD` or `include` statements.

# What else can I do?
Now that your apps only contain two lines of code, it's really easy to move them between environments.  You can simply copy and rename the app to get it to run in a different "environment".  Just make sure you've also copied the scripts to the appropriate sub-directory.  You can also use git to check out a specific tag/release in an environment.
