# Database Backup Script

This script automates the process of backing up PostgreSQL databases and copying the backups to a remote server. It also rotates older backups to manage storage space.

## Usage

To use this script, follow these steps:

1. Ensure that Docker-Compose is installed and configured properly on the system where the script will run.
2. Set up SSH keys for passwordless authentication to the remote server.
3. Create a `.db-backup.env` file in the `PROJECT_DIR` directory with the necessary environment variables (see below for details).
4. Modify the script to adjust any parameters or commands specific to your environment.
5. Run the script manually or schedule it as a cron job for automated backups.

## Environment Variables

The script relies on the following environment variables, which should be defined in the `.db-backup.env` file:

- `REMOTE_USER`: Username for SSH authentication to the remote server.
- `REMOTE_HOST`: Hostname or IP address of the remote server.
- `REMOTE_DESTINATION_DIRECTORY`: Directory on the remote server where backups will be copied.
- `BACKUP_DIR`: Directory whose contents are being archived into the tar file.
- `DISCORD_WEBHOOK_URL`: Webhook URL for Discord notifications.
- `MAX_BACKUPS`: Number of latest backups to store.
- `SUCCESS_MESSAGE_DISCORD_USER_IDS`: Discord user IDs to tag in success messages (space-separated string, e.g., "userID1 userID2").
- `FAILURE_MESSAGE_DISCORD_USER_IDS`: Discord user IDs to tag in failure messages (space-separated string, e.g., "userID1 userID2").

## Script Workflow

1. **Load Environment Variables**: Load environment variables from the `.db-backup.env` file.
2. **Check Environment**: Ensure that all required environment variables are set and not empty.
3. **Dump Databases**: Use Docker-Compose to dump PostgreSQL databases to local files.
4. **Create Tar Archive**: Create a tar archive of the backup directory.
5. **Copy Backup Files**: Copy the tar.gz file to the remote server using SCP.
6. **Remove Local Backup**: Remove the local tar.gz file to save disk space.
7. **Rotate Backups**: Remove older backups on the remote server to manage storage space.
8. **Record Time and Cleanup**: Record start and end times, calculate total time taken, and log the results.

## Error Handling

- If any step fails (e.g., backup creation, SCP), the script exits with an error message and notifies via Discord webhook.
- Rotation of older backups on the remote server may also fail, in which case an error message is logged.

## Cron Job Execution

To schedule this script as a cron job for regular backups, add an entry to the crontab file with the desired frequency (e.g., every day, every week). Make sure to set up the cron environment properly to include all dependencies and environment variables.

Example crontab entry to run the script every day at 2:00 AM:

```bash
0 2 * * * /path/to/your/script.sh
```

## Logging and Debugging

For debugging purposes, you can redirect output to a log file by modifying the script to append `>> /path/to/logfile.log 2>&1` to each command. This will capture both standard output and error output.

```bash
0 2 * * * /path/to/your/script.sh >> /path/to/logfile-name.log 2>&1
```

## Dependencies

- Docker-Compose
- SSH
- Cron (if running as a cron job)

## License

This script is provided under the [MIT License](https://opensource.org/licenses/MIT).s