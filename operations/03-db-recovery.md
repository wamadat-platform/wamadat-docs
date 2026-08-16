# DB Recovery Runbook

**Purpose:** restore Wamadat from backup in real disaster scenarios. Drill-validated 2026-05-14.

## When you'd run this

| Scenario | Source |
|---|---|
| Accidental DROP / TRUNCATE / DELETE | `backups/` (local, last 7) |
| Disk corruption (DB unreadable) | `OneDrive\Wamadat-Backups\` (last 30) |
| Machine lost / stolen / dead | OneDrive (sync to replacement machine first) |
| Bad migration in production | Most recent backup BEFORE the migration ran |

## Pre-flight (always do these first)

1. **Stop all writes** — kill `php artisan queue:work`, stop `php artisan serve`, stop frontend, log out users via maintenance mode:
   ```powershell
   cd C:\Users\U\wamadat-platform\backend
   php artisan down --message="Database recovery in progress" --retry=300
   ```

2. **Identify the backup** to restore from. Newest is in `backups\` locally:
   ```powershell
   Get-ChildItem C:\Users\U\wamadat-platform\backups -Filter 'wamadat-backup-*.zip' |
       Sort-Object LastWriteTime -Descending | Select-Object -First 5
   ```

3. **Confirm with operator** which timestamp. NEVER guess.

## Full restore procedure

```powershell
# 1. Extract the chosen backup
$Stamp   = '20260514-224200'   # change to chosen timestamp
$Archive = "C:\Users\U\wamadat-platform\backups\wamadat-backup-$Stamp.zip"
$Work    = "$env:TEMP\wamadat-restore-$Stamp"
Expand-Archive -Path $Archive -DestinationPath $Work -Force

# 2. Set password (read from backend/.env value)
$env:PGPASSWORD = '<paste DB_PASSWORD from backend\.env>'
$PgBin = 'C:\Program Files\PostgreSQL\16\bin'

# 3. Rename current DBs as forensic copies (NEVER drop them — investigation may need them)
& "$PgBin\psql.exe" -h 127.0.0.1 -U postgres -d postgres -c "ALTER DATABASE wamadat_landlord RENAME TO wamadat_landlord_before_$Stamp;"
& "$PgBin\psql.exe" -h 127.0.0.1 -U postgres -d postgres -c "ALTER DATABASE wamadat_tenants  RENAME TO wamadat_tenants_before_$Stamp;"

# 4. Create fresh target DBs
& "$PgBin\psql.exe" -h 127.0.0.1 -U postgres -d postgres -c "CREATE DATABASE wamadat_landlord;"
& "$PgBin\psql.exe" -h 127.0.0.1 -U postgres -d postgres -c "CREATE DATABASE wamadat_tenants;"

# 5. Restore globals + data
& "$PgBin\psql.exe"       -h 127.0.0.1 -U postgres -d postgres          -f "$Work\globals.sql"
& "$PgBin\pg_restore.exe" -h 127.0.0.1 -U postgres -d wamadat_landlord  --no-owner --no-acl "$Work\landlord.dump"
& "$PgBin\pg_restore.exe" -h 127.0.0.1 -U postgres -d wamadat_tenants   --no-owner --no-acl "$Work\tenants.dump"

# 6. Sanity check
& "$PgBin\psql.exe" -h 127.0.0.1 -U postgres -d wamadat_landlord -c "SELECT COUNT(*) FROM tenants; SELECT COUNT(*) FROM plans;"
& "$PgBin\psql.exe" -h 127.0.0.1 -U postgres -d wamadat_tenants  -c "\dn tenant_*"

# 7. Bring app back up
cd C:\Users\U\wamadat-platform\backend
php artisan up
# Restart queue worker (NSSM service or manual)
```

## Restoring from offsite (machine lost)

1. Install PostgreSQL 16 + git on the new machine.
2. `git clone https://github.com/asseerimishal-coder/wamadat-platform.git C:\Users\U\wamadat-platform`
3. Configure `backend\.env` with the same DB credentials.
4. Sign in to OneDrive on the new machine — `Wamadat-Backups` folder syncs down.
5. Copy the most recent `.zip` from OneDrive into `C:\Users\U\wamadat-platform\backups\`.
6. Follow the **Full restore procedure** above starting at step 1.
7. Re-install NSSM and re-register the queue worker service (`docs/operations/04-worker-supervision.md`).

## Forensic copies — when to drop

The renamed `_before_<stamp>` DBs are kept until:
- The restored DB is verified by the operator (queries return expected data, app works end-to-end).
- A new backup is taken AFTER the restore (proves the new state is durable).
- Minimum 7 days have passed.

Then:
```powershell
& "$PgBin\psql.exe" -h 127.0.0.1 -U postgres -d postgres -c "DROP DATABASE wamadat_landlord_before_<stamp>;"
& "$PgBin\psql.exe" -h 127.0.0.1 -U postgres -d postgres -c "DROP DATABASE wamadat_tenants_before_<stamp>;"
```

## Drill schedule

- **Weekly Sunday**: operator runs `scripts\operations\restore-drill.bat`. Result PASS/FAIL appended to `restore-drill-log.txt`.
- **Any drift in row counts** → drill fails → investigate before next backup is trusted.
- **Quarterly**: operator runs the **full restore procedure** against a scratch DB pair (not _drill — uses real DB names with _restore_test suffix) to verify the runbook itself is still accurate.
