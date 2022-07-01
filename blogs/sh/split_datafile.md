title: Split data file in Shell, Bash or Command Prompt  
description: Split data file in Shell, Bash or Command Prompt  
keywords: shell, bash, sh, split, size, console, aws, files, tips, cheat, shell command, bash command, script, shell script, sql, teradata, redshift, vba, database, data warehouse, etl, custom, big data

#Split data file in Shell or Bash
##Split file into equal size
```console
filename=<your_data_file_full_path>
# e.g. filename=/data/orders.tbl
split --lines $(( $(expr $(expr 1024 * 1024 * 1024 * 1) / $(wc -L < ${filename})) )) ${filename} ${filename}.

```

##Split file into number of files
```console

split--lines $(wc -l <input file> | awk '{printf("%.0f", $1/<num_of_files>)}') <input file> <prefix>.


```
