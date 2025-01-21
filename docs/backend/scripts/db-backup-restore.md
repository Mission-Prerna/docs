# Database Backup/Restore Procedure

This document outlines the steps to backup and restore a database to the NL server and update test backups on staging.

## 1. Download the Database Dump

Run the following command to download the database dump file:
```bash
nohup sudo wget "<db-dump-url>" -O <file-name>.tar.gz &
```

- `nohup` is used to run the command in the background, ensuring it continues running if the terminal is closed.
- Check logs in nohup log:
  ```bash
  cat <path>/nohup.out 
  ```
- Check disk usage summary:
  ```bash
  du -sh *
  ```
- Check disk usage progress summary (as a root user):
  ```bash
  watch -n1 du -s *
  ```

## 2. Extract the ZIP File

Run the following command to extract the downloaded zip file:
```bash
sudo nohup tar -xvzf <file-name>.tar.gz
```

- Check logs in nohup log:
  ```bash
  cat <path>/nohup.out 
  ```
- Check disk usage summary:
  ```bash
  sudo du -sh *
  ```
- Check disk usage progress summary (as a root user):
  ```bash
  watch -n1 du -s *
  ```

## 3. Create Docker Compose

Navigate to the folder where the unzipped folders exist.

Create a `docker-compose.yml` file there and add Docker Compose configuration codes.

Run the following command to start the Docker containers:
```bash
docker-compose up -d
```

## 4. Access Docker Container

Run the following command to access the Docker container:
```bash
docker-compose exec tsdb bash
```

## 5. Restore the Database Dump

Follow the steps outlined in the documentation [here](https://docs.timescale.com/migrate/latest/pg-dump-and-restore/pg-dump-restore-from-timescaledb/#restore-into-the-target-database).

or after ensuring that the correct TimescaleDB version is installed you can go alternatively with below command

Run the following command to restore the database dump:
```bash
nohup psql -U "<user-name>" -d "<DB-name>" -v ON_ERROR_STOP=1 --echo-errors -f roles.sql -c "SELECT public.timescaledb_pre_restore();" -f dump.sql -c "SELECT public.timescaledb_post_restore();" &
```

If facing issues such as `role <user> already exists` and restore script is failing, edit the `roles.sql` file and comment out the `CREATE ROLE <user>` command before proceeding.