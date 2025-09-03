# Collation Version Mismatch

## Issue
  This was detected when running postgres on as container on the 
  K3S cluster. The error message was:
  
  ```json
{"@timestamp":"2025-09-03T20:51:45.793739479Z",
  "@version":"1",
  "message":"database \"dev_db\" has a collation version mismatch",
  "logger_name":"org.hibernate.engine.jdbc.spi.SqlExceptionHelper",
  "thread_name":"http-nio-8080-exec-6",
  "level":"WARN",
  "level_value":30000,
  "application":"RsvpBackend"}
  ```

## Root Cause Analysis
  After some research it appears that this is caused by a mismatch between the collation version of the database and the collation version of the operating system. 
  This can happen when the operating system is upgraded and the collation version is changed.

  The collation version is stored in the database and is used to determine how strings are compared and sorted. 
  If the collation version of the database does not match the collation version of the operating system, 
  then string comparisons and sorts may not work as expected.

  The theory is for this is that the container image used to run Postgres on the K3S cluster was updated to a newer version that uses a different collation version than the version used to create the database.
  
## Solution 
   For the solution I port forwarded the database to my local machine, then using Datagrip I ran:
   
    ```sql
    ALTER DATABASE dev_db REFRESH COLLATION VERSION;
    REINDEX DATABASE dev_db;
    ```
## Verification
After applying the solution, verify the issue is resolved by:
1. load a guests link, forcing the app to make a database call.
2. in explorer in grafana run the following query:
   ```sql
   count(rate({service_name="rsvp-backend"} |= `has a collation version mismatch` [$__auto]))
   ```
   and verify the count is 0.
