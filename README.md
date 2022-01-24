# backup-rclone

This is a python script to manage profiles for [`rclone`](https://rclone.org/).

_Simple, no dependencies, plain Python 3.5._

You can specify all settings for a `rclone` profile, source, destination, pre/post executable commands, extra flags, etc. It is possible to run a specific profile or all profiles at once(When no profile is specified).

The `source`/`destination` are remotes in `rclone` configuration, they must already exist. Use `rclone config` to create/edit.

## Table of Contents

* [Usage](#usage)
* [Profiles](#profiles)
    * [Options](#options)
    * [Fast List](#fast-list)
    * [Filters](#filters)
* [Installation](#installation)
* [Examples](#examples)
* [License](#license)

## Usage

```
usage: backup-rclone [-h] [-v] [-c CONFIG_FILE] [-l LOG_FILE] [-s] [--check] [-p PROFILE] [-e GLOBAL_PRE_EXEC] [-o GLOBAL_POST_EXEC] [--version]

Create Rclone backups

optional arguments:
  -h, --help            show this help message and exit
  -v, --verbose         Verbose, can be issued multiple times to increase verbosity(Max 3 times) (default: 0)
  -c CONFIG_FILE, --config-file CONFIG_FILE
                        Configuration file (default: /etc/backup-rclone.conf)
  -l LOG_FILE, --log-file LOG_FILE
                        Log file (default: /var/log/backup-rclone.log)
  -s, --show-profiles   Show profiles defined in the config file. (default: False)
  --check               Check the configuration file. (default: False)
  -p PROFILE, --profile PROFILE
                        Profile to run. Can be specified multiple times to run multiple profiles. If not provided, all profiles will be run (default: [])
  -e GLOBAL_PRE_EXEC, --global-pre-exec GLOBAL_PRE_EXEC
                        Command to be executed before the profile is run (default: None)
  -o GLOBAL_POST_EXEC, --global-post-exec GLOBAL_POST_EXEC
                        Command to be executed after the profile is run (default: None)
  --version             show program's version number and exit
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
* `disable_fast_list`: Disable the `--fast-list` flag in some remote types. See Fast List below for more info.
* `extra_options`: Provide others flags to `rclone`.
* `filter`: See Filters below.
* `pre_exec`: Command to be executed before the profile is run. Do not confuse with the `--global-pre-exec` which will be executed independently of the profiles.
* `post_exec`: Command to be executed after the profile is run. Do not confuse with the `--global-post-exec` which will be executed independently of the profiles.

### Fast List

In some remotes, the parameter `--fast-list` is advisable, to reduce costs, `backup-rclone` adds it automatically. The option `disable_fast_list` will override this behaviour, not adding this flag in those remotes.

Remotes that have `--fast-list` added automatically:

* `s3`
* `b2`
* `google cloud storage`
* `drive`
* `hubic`
* `jottacloud`
* `azureblob`
* `swift`
* `qingstor`

### Filters

Filters are exactly the same as `rclone` specification, see [Filtering](https://rclone.org/filtering/). One filter per line, see Examples below.

Filters starts with `-` or `+`, `-` will exclude the pattern from the list, `+` will add. If `filter` is not provided, the default will be used, `+ /**`, which will include all files.

## Installation

Currently, there is no release or version number, so the easiest installation procedure is to download the script and make it executable:

```bash
curl "https://raw.githubusercontent.com/0x3333/backup-rclone/master/backup-rclone" \
    > /usr/bin/backup-rclone && chmod +x /usr/bin/backup-rclone && /usr/bin/backup-rclone --version
```

The `backup-rclone` executable will be available in the `/usr/bin` folder, and is ready to go.

I suggest using [Chronic](https://github.com/docwhat/chronic) when using `cron`, it will handle the output in case of failure and you will be notified only when an error has ocurred. Something like this:

```
0 0 * * * chronic /usr/bin/backup-rclone
```

You can even use verbose output, as it will only be sent if an error occurs:

```
0 0 * * * chronic /usr/bin/backup-rclone -vvv
```

### Logrotate

It's highly recommended to configure `logrotate` to rotate the log files. A configuration is provided in the repo, `logrotate.conf`, just copy to `/etc/logrotate.d` named `backup-rclone` and you are good to go.

```bash
curl "https://raw.githubusercontent.com/0x3333/backup-rclone/master/logrotate.conf" \
    > /etc/logrotate.d/backup-rclone
```

## Examples

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

## License

`backup-rclone` is primarily distributed under the terms of both the MIT license and the Apache License (Version 2.0).

See LICENSE-APACHE and LICENSE-MIT for details.
