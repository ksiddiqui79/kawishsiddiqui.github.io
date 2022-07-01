title: Turn off FallBackSetting in Teradata
description: Turn off FallBackSetting in Teradata
keywords: utility, vacuum, analyze, data lake, datalake, formation, case study, shell, bash, sh, split, size, console, aws, files, tips, cheat, shell command, bash command, script, shell script, sql, teradata, redshift, vba, database, data warehouse, etl, custom, big data
#Turn off FallBackSetting in Teradata
**Below are the steps to turn off FallBackSettings “ALWAYS FALLBACK” in Teradata Server on-Prem or cloud (Amazon EC2 or Azure)**

* Login to Teradata NodeLogin to Teradata Node and run `sudo su –`
* Run command `dbscontrol` from command prompt
* In DBSControl window
	* Run `mod systemfe=T` and enter `Y`on prompt
	* Run `display internal` and note FallBackSetting parameter number
	* Run `modify internal <FallBackSetting parameter number>=1`  
e.g. `modify internal 113=1`
	* Run `write`
	* Run `quit`
* Restart database from command prompt:
	* `/etc/init.d/tpa stop`
	* `/etc/init.d/tpa start`

