# backup-rclone

This is a python script to manage profiles for [rclone](https://rclone.org/).

You can specify all settings, source, destination for a given profile. It is possible to run a specific profile or all profiles at once.

The `source`/`destination` remotes must already exists on rclone config. Use `rclone config` to create/edit.

## Usage

```
usage: backup-rclone [-h] [-v] [-c CONFIG_FILE] [-l LOG_FILE] [-p PROFILE]

Create Rclone backups

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         Verbose, can be issued multiple times to increase
                        verbosity(Max 3 times) (default: 0)
  -c CONFIG_FILE, --config-file CONFIG_FILE
                        Configuration file (default: /etc/backup-rclone.conf)
  -l LOG_FILE, --log-file LOG_FILE
                        Log file (default: /var/log/backup-rclone.log)
  -p PROFILE, --profile PROFILE
                        Profile to run (default: None)
```

***Note: If a profile is not provided, all profiles will be run.***

## Profiles

Profiles are configured using `/etc/backup-rclone.conf` file or provide a file using `--config-file` flag, this file has a structure similar to Windows INI files, it is parsed using Python's [`configparser`](https://docs.python.org/3.7/library/configparser.html) module.

### Options

* `[PROFILE-NAME]`: The profile name to be used with flag `--profile`.
* `action`: The action to be executed in `rclone`, can be [`copy`](https://rclone.org/commands/rclone_copy/) or [`sync`](https://rclone.org/commands/rclone_sync/).
* `source`: The remote to be used as source, empty if local folder.
* `source_path`: The source folder in the remote or local.
* `destination`: The remote to be used as destination, empty if local folder.
* `destination_path`: The destination folder in the remote or local.
* `disable_fast_list`: See Fast List below.
* `extra_options`: Provide others flags to `rclone`.
* `filter`: See Filters below.

### Fast List

In some remotes, the parameter --fast-list is advisable, to reduce costs, backup-rclone adds it automatically, this option disable this behavour in those remotes.

Remotes affected by this options:

* s3
* b2
* google cloud storage
* drive
* hubic
* jottacloud
* azureblob
* swift
* qingstor

### Filters

Filters are exactly the same as `rclone` specification, see [Filtering](https://rclone.org/filtering/). One filter per line, see Examples below.

Filters starts with `-` or `+`, `-` will exclude the pattern from the list, `+` will add. If `filter` is not provided, the default will be used, `+ /**`, which will include all files.

### Examples

```ini
[profile-name-b2]
action = sync
source_path = /Users/myuser
destination = b2-bucket-name
destination_path = /backups/mymachine/myuser
disable_fast_list = false
extra_options = --tpslimit 4 --transfers 4 --copy-links --delete-excluded --b2-hard-delete
filter =    - Thumbs.db
            - .DS_Store
            - ._*
            - ~$*
            - /.npm/**
            + /**

[profile-name-2]
action = copy
source_path = /root
destination = onedrive-x
destination_path = /
extra_options = --verbose
```

# License

This is free software under the terms of MIT the license (check the LICENSE file included in this package).
