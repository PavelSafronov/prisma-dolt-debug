# Dolt Prisma bug

## Installing

1. install package dependencies
    - `pnpm install`
2. install dolt by following [these directions](https://docs.dolthub.com/introduction/installation)
3. install MySQL by following [these directions](https://dev.mysql.com/doc/mysql-installation-excerpt/5.7/en/)
    - MySQL is being used during the test step (`npm run test`) to drop and recreate obsidian DB
    - you can skip MySQL installation and handle creating/dropping obsidian DB manually
    - if you do, remove the script "pretest" from package.json

## Happy path

1. Start MySQL
    - `mysql.server restart`
2. Start Prisma
    - `npm run test`
3. Migration should be successful

## Unhappy path

1. Start dolt
    - `dolt sql-server --loglevel=debug`
2. Start Prisma
    - `npm run test`
3. Migration should fail with the following message:
    - ```
      ...
      Error: Error: unknown error: table not found: _prisma_migrations
        0: migration_core::state::DevDiagnostic
                  at migration-engine/core/src/state.rs:251
      ...
      ```

## Logs

DB logs for both MySQL (works) and dolt (Prisma throws error) are in this repo: [mysql-logs.txt](mysql-logs.txt) and [dolt-logs.txt](dolt-logs.txt).

### Analysis?

Here is an excerpt from the dolt logs, right before Prisma throws its error:
```
2022-10-13T10:21:01-07:00 INFO [conn 5] NewConnection {DisableClientMultiStatements=false}
2022-10-13T10:21:01-07:00 DEBUG [conn 5] Starting query {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@socket}
2022-10-13T10:21:01-07:00 DEBUG [conn 5] Query finished in 0 ms {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@socket}
2022-10-13T10:21:01-07:00 INFO [conn 6] NewConnection {DisableClientMultiStatements=false}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Starting query {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@max_allowed_packet}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Query finished in 0 ms {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@max_allowed_packet}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Starting query {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@wait_timeout}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Query finished in 0 ms {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@wait_timeout}
2022-10-13T10:21:01-07:00 INFO [conn 5] ConnectionClosed {}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Starting query {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@max_allowed_packet}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Query finished in 0 ms {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@max_allowed_packet}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Starting query {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@wait_timeout}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Query finished in 0 ms {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@wait_timeout}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Starting query {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@version, @@GLOBAL.version}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Query finished in 1 ms {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@version, @@GLOBAL.version}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Starting query {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@lower_case_table_names}
2022-10-13T10:21:01-07:00 DEBUG [conn 6] Query finished in 0 ms {connectTime=2022-10-13T10:21:01-07:00, connectionDb=obsidian, query=SELECT @@lower_case_table_names}
2022-10-13T10:21:01-07:00 INFO [no conn] kill query: pid 25 {}
2022-10-13T10:21:01-07:00 INFO [no conn] kill query: pid 27 {}
```

It looks like Prisma made some queries to get version and timeout info, but there are no calls to get a list of tables. But Prisma still fails with a `table not found: _prisma_migrations` error. Strange.
