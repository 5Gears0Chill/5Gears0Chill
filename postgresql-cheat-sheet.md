# PostgreSQL CLI Cheat Sheet

## System & Service Management

### Check if PostgreSQL is Running
```bash
# Check PostgreSQL service status (Linux/macOS)
sudo systemctl status postgresql
# or
brew services list | grep postgresql  # macOS with Homebrew

# Check if PostgreSQL process is running
ps aux | grep postgres
# or
pgrep -f postgres

# Check if PostgreSQL is listening on port 5432
netstat -an | grep 5432
# or
lsof -i :5432

# Test connection to PostgreSQL
pg_isready
# or with specific host/port
pg_isready -h localhost -p 5432
```

### Start/Stop PostgreSQL Service
```bash
# Start PostgreSQL service
sudo systemctl start postgresql    # Linux
brew services start postgresql     # macOS

# Stop PostgreSQL service
sudo systemctl stop postgresql     # Linux
brew services stop postgresql      # macOS

# Restart PostgreSQL service
sudo systemctl restart postgresql  # Linux
brew services restart postgresql   # macOS

# Enable PostgreSQL to start at boot
sudo systemctl enable postgresql   # Linux
```

## Connection & Authentication

### Connecting to PostgreSQL
```bash
# Connect as default user to default database
psql

# Connect as specific user
psql -U username

# Connect to specific database
psql -d database_name

# Connect to remote server
psql -h hostname -p 5432 -U username -d database_name

# Connect with connection string
psql "postgresql://username:password@hostname:5432/database_name"

# Connect and run single command
psql -U username -d database_name -c "SELECT version();"

# Connect and run SQL file
psql -U username -d database_name -f script.sql
```

### Connection Examples
```bash
# Common connection patterns
psql -U postgres                           # Connect as postgres user
psql -U myuser -d mydb                     # Connect as myuser to mydb
psql -h localhost -U postgres -d template1 # Connect to localhost
psql postgres://postgres@localhost/mydb    # Using connection URI
```

## Database Management

### Database Operations
```bash
# Create database
createdb database_name
createdb -U username database_name

# Drop database
dropdb database_name
dropdb -U username database_name

# List all databases
psql -l
# or from within psql
\l

# Get database size
psql -d database_name -c "SELECT pg_size_pretty(pg_database_size('database_name'));"
```

### User Management
```bash
# Create user
createuser username
createuser -P username  # Prompt for password
createuser -s username  # Create superuser

# Drop user
dropuser username

# List users
psql -c "\du"
```

## Backup & Restore

### Database Backup
```bash
# Backup single database
pg_dump database_name > backup.sql
pg_dump -U username database_name > backup.sql

# Backup with custom format (smaller, faster restore)
pg_dump -Fc database_name > backup.dump

# Backup specific tables
pg_dump -t table_name database_name > table_backup.sql

# Backup all databases
pg_dumpall > all_databases.sql

# Backup with compression
pg_dump database_name | gzip > backup.sql.gz
```

### Database Restore
```bash
# Restore from SQL file
psql database_name < backup.sql

# Restore custom format
pg_restore -d database_name backup.dump

# Restore and create database
createdb new_database
psql new_database < backup.sql

# Restore with specific user
psql -U username -d database_name < backup.sql
```

## psql Interactive Commands

### Navigation & Information
```sql
-- List databases
\l

-- Connect to database
\c database_name

-- List tables
\dt

-- List all relations (tables, views, sequences)
\d

-- Describe table structure
\d table_name

-- List users/roles
\du

-- List schemas
\dn

-- Show current database and user
\conninfo

-- Show current database
SELECT current_database();

-- Show current user
SELECT current_user;
```

### Query & Display
```sql
-- Show query execution time
\timing

-- Show expanded display (vertical format)
\x

-- Save query results to file
\o filename.txt

-- Execute commands from file
\i filename.sql

-- Show last command
\g

-- Clear screen
\! clear

-- Quit psql
\q
```

## Monitoring & Performance

### System Information
```bash
# Show PostgreSQL version
psql -c "SELECT version();"

# Show server uptime
psql -c "SELECT current_timestamp - pg_postmaster_start_time() as uptime;"

# Show database sizes
psql -c "SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database ORDER BY pg_database_size(datname) DESC;"

# Show largest tables
psql -d database_name -c "
SELECT schemaname,tablename,pg_size_pretty(size) as size, pg_size_pretty(total_size) as total_size
FROM (
  SELECT schemaname,tablename,
    pg_relation_size(schemaname||'.'||tablename) as size,
    pg_total_relation_size(schemaname||'.'||tablename) as total_size
  FROM pg_tables
) AS TABLES
ORDER BY total_size DESC LIMIT 10;"
```

### Active Connections
```sql
-- Show active connections
SELECT pid, usename, datname, client_addr, state, query_start, query
FROM pg_stat_activity
WHERE state = 'active';

-- Count connections per database
SELECT datname, count(*) 
FROM pg_stat_activity 
GROUP BY datname;

-- Kill specific connection
SELECT pg_terminate_backend(pid) WHERE pid = 12345;

-- Kill all connections to a database
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'database_name' AND pid <> pg_backend_pid();
```

## Configuration & Logs

### Configuration Files
```bash
# Find PostgreSQL config file location
psql -c "SHOW config_file;"

# Find data directory
psql -c "SHOW data_directory;"

# Find log file location
psql -c "SHOW log_directory;"

# Show specific setting
psql -c "SHOW shared_buffers;"

# Show all settings
psql -c "SELECT name, setting FROM pg_settings ORDER BY name;"
```

### View Logs
```bash
# Find log files (common locations)
sudo find /var/log -name "*postgres*" 2>/dev/null
sudo find /usr/local/var/log -name "*postgres*" 2>/dev/null

# View recent log entries
sudo tail -f /var/log/postgresql/postgresql-*.log

# View logs using journalctl (systemd systems)
sudo journalctl -u postgresql -f
```

## Common Troubleshooting

### Connection Issues
```bash
# Check if PostgreSQL is accepting connections
pg_isready -h localhost -p 5432

# Check listen addresses
psql -c "SHOW listen_addresses;"

# Check port
psql -c "SHOW port;"

# Test local connection
psql -h localhost -U postgres

# Check authentication method
# Look at pg_hba.conf file location:
psql -c "SHOW hba_file;"
```

### Performance Issues
```sql
-- Show slow queries
SELECT query, mean_time, calls, total_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- Show locks
SELECT * FROM pg_locks;

-- Show blocking queries
SELECT blocked_locks.pid AS blocked_pid,
       blocked_activity.usename AS blocked_user,
       blocking_locks.pid AS blocking_pid,
       blocking_activity.usename AS blocking_user,
       blocked_activity.query AS blocked_statement
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity
  ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_locks blocking_locks
  ON blocking_locks.locktype = blocked_locks.locktype
  AND blocking_locks.DATABASE IS NOT DISTINCT FROM blocked_locks.DATABASE
  AND blocking_locks.relation IS NOT DISTINCT FROM blocked_locks.relation
JOIN pg_catalog.pg_stat_activity blocking_activity
  ON blocking_activity.pid = blocking_locks.pid
WHERE NOT blocked_locks.GRANTED;
```

## Quick Reference Examples

### Daily Operations
```bash
# Quick health check
pg_isready && echo "PostgreSQL is running"

# Quick backup
pg_dump myapp_production > "backup_$(date +%Y%m%d_%H%M%S).sql"

# Quick restore to development
dropdb myapp_development
createdb myapp_development
psql myapp_development < backup_20231201_143022.sql

# Check database size
psql -d myapp -c "SELECT pg_size_pretty(pg_database_size(current_database()));"

# List active connections
psql -c "SELECT count(*) FROM pg_stat_activity;"
```

### Environment Variables
```bash
# Set default connection parameters
export PGHOST=localhost
export PGPORT=5432
export PGUSER=myuser
export PGDATABASE=mydatabase
export PGPASSWORD=mypassword

# Then simply use:
psql  # Will use above defaults
```

## Security Best Practices

```bash
# Create read-only user
psql -c "CREATE USER readonly_user WITH PASSWORD 'secure_password';"
psql -c "GRANT CONNECT ON DATABASE mydb TO readonly_user;"
psql -c "GRANT USAGE ON SCHEMA public TO readonly_user;"
psql -c "GRANT SELECT ON ALL TABLES IN SCHEMA public TO readonly_user;"

# Revoke privileges
psql -c "REVOKE ALL ON DATABASE mydb FROM username;"

# Change password
psql -c "ALTER USER username PASSWORD 'new_password';"
```
