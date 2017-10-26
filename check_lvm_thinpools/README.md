# check_lvm_thinpools
A plugin for checking the size of an LVM thinpool

## Usage
```
./check_lvm_thinpools [-w <warning threshold>] [-c <critical threshold>] [-f <file with lvs output>]
```

`-w` and `-c` are required and should be any number 0.00 - 100.00 and will reflect the percentage of the thinpool used.

`-f` is optional and will read from a file that has the correctly formatted lvs output.

## Notes
This script utilizes the `lvs` command, which requires root privileges. However, if you want to avoid allowing the nagios user to have root permissions, you can have the script read from a file.
For example, set a cronjob that runs every so often and writes the output of `lvs --noheadings --separator ":" -o lv_name,data_percent -S "data_percent >= 0` to a file. Then, have this script read from that file with the `-f` argument.
