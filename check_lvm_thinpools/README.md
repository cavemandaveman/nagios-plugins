# check_lvm_thinpools
A plugin for checking the size of an LVM thinpool

## Usage
```
./check_lvm_thinpools [-w <warning threshold>] [-c <critical threshold>]
```

`-w` and `-c` should be any number 0.00 - 100.00 and will reflect the percentage of the thinpool used.

## Notes
This script utilizes the `lvs` command, which requires root privileges.
