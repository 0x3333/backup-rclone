# backup-rclone

This is a python script to manage profiles for [rclone](https://rclone.org/). _Minimum Python version: 3.5._

You can specify all settings, source, destination for a given profile. It is possible to run a specific profile or all profiles at once.

The `source`/`destination` remotes must already exists on rclone config. Use `rclone config` to create/edit.

## Usage

```
usage: backup-rclone [-h] [-v] [-c CONFIG_FILE] [-l LOG_FILE] [-p PROFILE]
                     [-e GLOBAL_PRE_EXEC] [-o GLOBAL_POST_EXEC]

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
                        Profile to run. Can be specified multiple times to run
                        multiple profiles. If not provided, all profiles will
                        be run (default: None)
  -e GLOBAL_PRE_EXEC, --global-pre-exec GLOBAL_PRE_EXEC
                        Command to be executed before the profile is run
                        (default: None)
  -o GLOBAL_POST_EXEC, --global-post-exec GLOBAL_POST_EXEC
                        Command to be executed after the profile is run
                        (default: None)
```

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
* `pre_exec`: Command to be executed before the profile is run. Do not confuse with the `--global-pre-exec` which will be executed independently of the profiles.
* `post_exec`: Command to be executed after the profile is run. Do not confuse with the `--global-post-exec` which will be executed independently of the profiles.

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

See some other examples in `examples` folder.

All possible options demonstrated:

```ini
[profile-name-b2]
# *Will* delete from the destination folder
action = sync
# If local folder, `source` is not necessary
source_path = /Users/myuser
# `b2-bucket-name` is the rclone remote name
destination = b2-bucket-name
destination_path = /backups/mymachine/myuser
# Will enable fast_list, which in B2 case, will reduce API usage
disable_fast_list = false
# This script(/usr/bin/my_script) will be executed before this profile with argument mount
pre_exec = /usr/bin/my_script mount
# This script(/usr/bin/my_script) will be executed before this profile with argument umount
post_exec = /usr/bin/my_script umount
extra_options = --tpslimit 4 --transfers 4 --copy-links --delete-excluded
filter =    - Thumbs.db
            - .DS_Store
            - ._*
            - ~$*
            - /.npm/**
            + /**

[profile-name-onedrive]
# Will *not* delete from the destination folder
action = copy
# If local folder, `source` is not necessary
source_path = /root
# `onedrive-backup` is the rclone remote name
destination = onedrive-backup
destination_path = /
# Will disable stats and copy links as if they were files
extra_options = --stats 0 --copy-links
```

# License

`backup-rclone` is primarily distributed under the terms of both the MIT license and the Apache License (Version 2.0).

See LICENSE-APACHE and LICENSE-MIT for details.
