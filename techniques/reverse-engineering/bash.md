# Bash

## Using debug 

At the top of your bash script, enter the following lines. 

```bash
exec 5> debug_output.txt
BASH_XTRACEFD="5"
```

Execute the bash script with the -x flag. The debug output will be printed to debug\_output.txt

```text
bash -x ./[script.sh]
```

