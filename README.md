# DQe-c
<img src="images/DQe-c_logo_more_drip.png" width="120px" align="left" hspace="10" vspace="6"/> 
**DQe-c** is a data quality assessment tool for clinical data repositories. The main focus of DQe-c is evaluation of data completeness and visualization of the test results.


# Description
This version of DQe-c was built using Python from the first version of [DQe-c](https://github.com/data2health/DQe-c) (which was in R). The goal was to make each of the tests in DQe-c modular and to make customization and test additions easier to build.

# Installation
DQe-c system requirements
```
>= python 3.6
```

Install the git repository
```
git clone https://github.com/data2health/DQe-c-v2.git
cd DQe-c-v2
```
Install the dependency libraries
```
pip install -r requirements_pip.txt
```
OR
```
conda install --file requirements.txt
```

# Configuration and Using the Tool
## Setting up Database Connection
All of the required configution variables are set in config.json or a derivative of config.json. If you have more than one repository database/schema/server/etc it is recommended that you duplicate the config.json file (one for each of the unique configurations).

Each of the variables in the configuration file need to be filled in.
```
"DBMS": "Database Management System",
Currently supported and tested options:
    "postgresql"
    "sql server"
Currently supported but untested options:
    "oracle"
    "redshift"

"database": "name of the database",

"CDM": "Common Data Model",
Current supported options:
    "OMOPV5_0"  # OMOP CDM version 5.0
    "OMOPV5_2"  # OMOP CDM version 5.2
    "OMOPV5_3"  # OMOP CDM version 5.3
    "PCORI3"    # PCORI version 3
    "PCORI31"   # PCORI version 3.1

"schema": "what schema are your clinical tables stored under?",
"vocabulary schema": "what schema is your vocabulary stored under (if separate)?",

"Credentials": {
    "User": "username",
    "Password": "password"
},

"ConnectionDetails": {
    "Host": "ex. localhost",
    "Port": "ex. 8080",
    "Driver": "{ODBC Driver 13 for SQL Server}"
                This is only necessary for SQL Server
},

"Organization": "Name of your organization",

"Name": "Who are you?"
```
## Running DQe-c
After configuration, simply run.
```
python DQe-c.py
```
This defaults to assuming that the configuration file is config.json. To use a different config file, simply run.
```
python DQe-c.py -c /path/to/config_other.json
```

# Module Workflow
The current version of DQe-c is organized in a modular workflow that centers around the Query.py module which keeps track of the tests' output and database connection details.
## Modular high level workflow
![workflow](images/DQe-c-v2_workflow_two.png)

## Module descriptions
### config.json
Collects the credentials and connection details from the user. These are used to later query the database for the various tests as well as track which CDM is being used.

### Query.py
This module starts the process by loading the CDM reference file into the DQTBL object and initializing the database connection. This module serves as the "base" from which to run the different module tests. As more modules are created, they should be built to use this module.

### Diff.py
Diff tests for difference between the reference common data model and what is acutally present in the database. All tables and columns that are supposed to be present (part of the reference CDM) are reported in tablelist.csv as being either present or absent. DQTBL is changed to include only the tables and columns that are present in the database. This ensures that future SQL queries don't query non-existant tables.

### Missingingness.py
This module runs through the whole repository and calculates the percent missingness for each column of each table. We check for nonsense values (%, #, !, @, etc) as well as NULL values. This module outputs test results to the DQTBL object.

### Orphan.py
Checks for orphan keys, or foreign keys that are not present in the primary table. An example is to check that all the person_id values in the observation or measurement table are present in the person table. The results are output to DQTBL.

### Indicators.py
Calculates the percentage of patients that don't have key clinical indicators in their records. This includes measurements like Blood Pressure, Heart Rate, or White Blood Cell count. We also include overall record completeness checks, like what percent of patients don't have a visit, a medication, or an observation associated with their record. The output of this test is written to indicators.csv. Check the wiki to see how to add new clinical indicator checks.