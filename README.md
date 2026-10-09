# Week 6 - Backup, Recovery and Replication

## Step 1: Logical Backup
mkdir -p ~/backups
pg_dump -Fc -f ~/backups/bootcamp.dump bootcamp
pg_restore --list ~/backups/bootcamp.dump | head
createdb bootcamp_check && pg_restore -d bootcamp_check ~/backups/bootcamp.dump

## Step 2: WAL Archiving
wal_level = replica
archive_mode = on
archive_command = 'cp %p ~/backups/wal/%f'
mkdir -p ~/backups/wal
sudo systemctl restart postgresql
pg_basebackup -D ~/backups/base -Ft -z -Xs -P

## Step 3: PITR Disaster Recovery
SELECT now(); -- record: 2025-06-01 10:00:00
DELETE FROM students;
-- Recovery:
restore_command = 'cp ~/backups/wal/%f %p'
recovery_target_time = '2025-06-01 10:00:00'
SELECT count(*) FROM students;

## Step 4: Streaming Standby
CREATE ROLE replicator WITH REPLICATION LOGIN PASSWORD 'reppass';
-- in pg_hba.conf: host replication replicator 127.0.0.1/32 md5
pg_basebackup -h 127.0.0.1 -U replicator -D ~/standby -R -P

## Step 5: Monitor Replication
SELECT application_name, state, pg_wal_lsn_diff(sent_lsn, replay_lsn) FROM pg_stat_replication;
