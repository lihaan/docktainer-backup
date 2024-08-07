# docktainer-backup: Docker Container Filesystem Backups
Daily backups of specific files/directories within Docker containers, with Telegram-supported notifications


## Overview
```*record scratch* *freeze frame* ⁣ Yep, that's me. You're probably wondering how I ended up in this situation...```

My nightmare began when my Python Jupyter dev container had filled up my entire 1TB disk drive (WSL only recently enabled automatic shrinking of VHDs), causing the container to crash.

The Docker service wouldn't start due to lack of space, so I couldn't copy my files out. I couldn't make space or delete other files, since I had already transferred Docker to live in a separate drive its own. I found the virtual hard disk drive corresponding to docker-desktop-data, but had no idea how to open it to access its contents. I want to make a backup of it, but where am I going to find 1TB of free space at a moment's notice? As a last resort, I tried compressing the VHD via diskpart, and I managed to successfully compress it by 0B. Thank you Windows.

Hence, after losing about 2 weeks' worth of hard work, I decided to make a script to make daily backups of my containers. I wanted this system to be automated so I wouldn't forget, and to only make backups of specific directories where I do my development work (and not make unnecessary backups of python virtual environments). To further save on disk space, the script should perform the backup only when the container was run at least once ie. on days where I take a break from coding, no backups should be made. Also, automatic pruning!


## Features

#### Targeted file/directory backups
  - Paths to files/directories can be specified for each container ID, so that unnecessary contents do not take up disk space
#### Backup only when needed
  - Backups of specified paths are only made if the associated container had actually ran since its last backup
  - The minimum number of days between backups can also be configured
#### Automated Pruning
  - Backups of specified paths of deleted containers can be deleted when a user-configurable threshold duration is reached
  - Older backups can be deleted so that the total number of backups does not rise past a user-configurable value
#### Notifications via Telegram
  - I love Telegram so much that I incorporated disk usage and backup status updates to be delivered via a Telegram bot
  - Just specify your Telegram chat ID and your own Telegram bot token


## Video Demo

(to be updated!)


## Limitations

- Requires a separate scheduling service (ie. Task Scheduler, crontab) for automation
    - This script was made with the intention of being a "start-stop" program to avoid potential memory leaks / resource hogging behavior
    - There are definitely ways to mitigate this, but I'm constrained by time and (lack of) proficiency. Any pointers or suggestions would be great though!
- Finest granularity of daily backups
    - Technically, the script can still be run more than once daily, however any new backups created will overwrite its existing copy
    - Currently, daily backups already suit my own use case pretty well, but I might make this even more dynamic in the future


## Requirements

- Python 3
  - Tested: >= 3.10.4
- Docker (I'm hilarious I know)
  - Tested: version 24.0.6, build ed223bc


## Setup

1. Clone repository
   > \> git clone https://github.com/lihaan/docktainer-backup.git

   > \> cd docktainer-backup
   
2. Install dependencies
   - Recommended to create a virtual environment (eg. venv) before installing
      > \> pip install -r requirements.txt

3. (Optional) Obtain Telegram chat ID and bot token
   - [External link to instructions](https://www.alphr.com/find-chat-id-telegram/)
   - The method described in the link basically involves starting a conversation with the @RawDataBot on Telegram, which will return some (non-sensitive) information about your account ie. your chat ID.
  
4. Configure *config.yaml*
   - Parameters and their default options are listed in it, as well as at [Configuration Options](#configuration-options)

5. [Schedule](#configuration-options) the script!


## Schedule

### Via Windows Task Scheduler
1. Create a .cmd file (eg. script.cmd)
2. Insert the following text into it, replacing the content in \<arrow brackets\> with your own paths
    > <PATH_TO_YOUR_PYTHON_DIR>\pythonw.exe <PATH_TO_REPOSITORY_DIR>\docktainer-backup\container_backup.py

3. Create a task with Task Scheduler with script.cmd file as the action, that triggers once everyday
    - you may in fact run the script as often as you like based on your use case
    - see the configuration options for ways to 

### Via crontab
(Sorry I don't daily drive Linux)


## Configuration Options
#### Note:
- An "instance" refers to a specified path on a specific container
- Default value for the option will be used if the option is left out altogether

### Backup settings
#### min_backup_interval: 0 (int)
- Minimum no. of days between each instance's backups
- "Minimum": An instance will only be backed up after *min_backup_interval* full days has passed since its last backup, and if the corresponding container ran during that interval / is currently running
- Default value of 0 means that every time the script is run, if the container is currently running, a new backup will be made

#### ghost_backup_keep_days: -1 (int)
- Max number of days to keep backups of an instance after its container has been deleted
- Default value of -1 means such backups are never pruned

#### backup_keep_num: -1 (int)
- Max number of backups of an instance to keep
- Must be a non-zero positive number, or -1 (default value)
- Pruning of oldest backups occurs before new ones are created (but if pruning fails, the backup will still be made)
- Default value of -1 means old backups are never pruned

#### warn_large_backup_mb: 1024 (int)
- Warn if difference between newly created backup and pruned size is more than specified amount in mebibytes (MiB)

#### backup_by_default: True (bool)
- If True, backup all containers by default, else if False, only those specified in *container_paths*

#### container_paths: *blank* (dict - {str/numeric: str})
- Dictionary of container IDs (provide at least the first 12 characters), each with a list of paths to backup (see example provided)
- If *backup_by_default*=True, containers not listed in *container_paths* will have their entire filesystem backed up
- As the script does not keep track of changes over time, backups for all specified paths for a container will occur as long as either:
  1. the container has not been backed up before
  2. at least *min_backup_interval* days has passed since the last backup, AND the container had run at least once since then
- Removing a specified path will mark that corresponding instance as deleted
  - Conversely, an instance that has been marked as deleted can be restored if its container and path is added back to *container_paths* (and if its ghost backups have not been completely pruned)
- Example configuration:
```
container_paths:
  fd312f3n7h45:
  - /etc
  - /home/main.py
  container_id_2:
  - /path/to/file.ext
  - /path/to/dir
  ```


### Notification settings

#### telegram_chat_id: *blank*
- Chat ID of your telegram account to send notifications to (must be used together with *telegram_bot_token*)

#### telegram_bot_token: *blank*
- Bot Token of your telegram bot to send notifications from (must be used together with *telegram_chat_id*)


### Paths to directories containing output file (Ensure these are created already)

#### archive_dir_path: *current_directory* (str)
- Directory containing backup folder, log folder, and instance_info.csv (a persistent file that stores metadata about each instance and its backups)
- If specified value cannot be located, the script defaults to using this current folder as the archive destination


## Contributing

*docktainer-backup* is born from literal sweat and tears (mostly tears). Do consider [buying this struggling university student a coffee](https://www.buymeacoffee.com/lihanong)! Thank you!

~ Li Han
