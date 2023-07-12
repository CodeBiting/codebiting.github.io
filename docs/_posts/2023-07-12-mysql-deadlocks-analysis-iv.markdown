---
layout: post
title:  "MySQL Deadlocks Analysis - Part I - Documentation"
date:   2023-07-12 13:35:24 +0200
categories: development
---
Learn how deadlocks works in .

## Isolation

  In [database](https://en.wikipedia.org/wiki/Database "Database") systems, **isolation** determines how [transaction](https://en.wikipedia.org/wiki/Database_transaction "Database transaction") integrity is visible to other users and systems.

  A lower isolation level increases the ability of many users to access the same data at the same time, but increases the number of [concurrency](https://en.wikipedia.org/wiki/Concurrency_(computer_science) "Concurrency (computer science)") effects (such as [dirty reads](https://en.wikipedia.org/wiki/Write%E2%80%93read_conflict "Write–read conflict") or lost updates) users might encounter. Conversely, a higher isolation level reduces the types of concurrency effects that users may encounter, but requires more system resources and increases the chances that one transaction will block another.

  1. **Dirty reads**:
    A _dirty read_ (aka _uncommitted dependency_) occurs when a transaction retrieves a row that has been updated by another transaction that is not yet committed.

  2. **Non-repeatable reads**
    A _non-repeatable read_ occurs when a transaction retrieves a row twice and that row is updated by another transaction that is committed in between.

  3. **Phantom reads**
    A _phantom read_ occurs when a transaction retrieves a set of rows twice and new rows are inserted into or removed from that set by another transaction that is committed in between.

- **Intention Locks**:

    `InnoDB` supports _multiple granularity locking_ which permits coexistence of row locks and table locks. For example, a statement such as [`LOCK TABLES ... WRITE`](https://dev.mysql.com/doc/refman/8.0/en/lock-tables.html "13.3.6 LOCK TABLES and UNLOCK TABLES Statements") takes an exclusive lock (an `X` lock) on the specified table. To make locking at multiple granularity levels practical, `InnoDB` uses [intention locks](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_intention_lock "intention lock"). Intention locks are table-level locks that indicate which type of lock (shared or exclusive) a transaction requires later for a row in a table. There are two types of intention locks:
  
  - An [intention shared lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_intention_shared_lock "intention shared lock") (`IS`) indicates that a transaction intends to set a _shared_ lock on individual rows in a table.

  - An [intention exclusive lock](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_intention_exclusive_lock "intention exclusive lock") (`IX`) indicates that a transaction intends to set an exclusive lock on individual rows in a table.

## InnoDB

  InnoDB offers all four transaction isolation levels described by the SQL:1992 standard: [`READ UNCOMMITTED`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-uncommitted), [`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed), [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read), and [`SERIALIZABLE`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_serializable). The default isolation level for `InnoDB` is [`REPEATABLE READ`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_repeatable-read).
  
  1. **READ UNCOMMITTED**:
     [SELECT](https://dev.mysql.com/doc/refman/8.0/en/select.html "13.2.13 SELECT Statement") statements are performed in a nonlocking fashion, but a possible earlier version of a row might be used. Thus, using this isolation level, such reads are not consistent. This is also called a [dirty read](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_dirty_read "dirty read"). Otherwise, this isolation level works like [`READ COMMITTED`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-isolation-levels.html#isolevel_read-committed).
  
  2. **READ COMMITED**:
     A transaction can only read committed data. It ensures that any data read by a transaction has been committed by another transaction. However, it still allows non-repeatable reads and can result in phantom reads.
  
  3. **REPEATABLE READ**:
     This isolation level guarantees that if a transaction reads a set of data multiple times, it will always see the same data. It prevents non-repeatable reads, but phantom reads can still occur. In this level, any data read by a transaction is locked until the transaction completes.
  
  4. **SERIALIZABLE**:
     It provides strict transaction isolation, ensuring that multiple transactions executing concurrently will have the same effect as if they were executed sequentially. Serializable isolation eliminates all concurrency-related anomalies such as dirty reads, non-repeatable reads, and phantom reads. It achieves this by applying strict locking and ensuring that no other transactions can modify the data until the current transaction completes.
  
  How to **SET TRANSACTION** statement:

    ```sql
    SET [GLOBAL | SESSION] TRANSACTION
    transaction_characteristic [, transaction_characteristic] ...
    
    transaction_characteristic: {
    ISOLATION LEVEL level
      | access_mode
    }
    
    level: {
      REPEATABLE READ
      | READ COMMITTED
      | READ UNCOMMITTED
      | SERIALIZABLE
    }
    
    access_mode: {
      READ WRITE
      | READ ONLY
    }
    ```
  
- These statements provide control over use of [transactions](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_transaction "transaction"):
  - `START TRANSACTION` or `BEGIN` start a new transaction (autocommit remains disabled until you end the transaction with `COMMIT` or `ROLLBACK`).

  - `COMMIT` commits the current transaction, making its changes permanent.

  - `ROLLBACK` rolls back the current transaction, canceling its changes.

  - `SET autocommit` disables or enables the default autocommit mode for the current session.
  
- Disabling autocommit mode by setting the [`autocommit`](https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html#sysvar_autocommit) variable to zero, changes to transaction-safe tables (such as those for [`InnoDB`](https://dev.mysql.com/doc/refman/8.0/en/innodb-storage-engine.html "Chapter 15 The InnoDB Storage Engine") or [`NDB`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster.html "Chapter 23 MySQL NDB Cluster 8.0")) are not made permanent immediately. You must use [`COMMIT`](https://dev.mysql.com/doc/refman/8.0/en/commit.html "13.3.1 START TRANSACTION, COMMIT, and ROLLBACK Statements") to store your changes to disk or `ROLLBACK` to ignore the changes.
  
- Beginning a transaction causes any pending transaction to be committed, also causes table locks acquired with [`LOCK TABLES`](https://dev.mysql.com/doc/refman/8.0/en/lock-tables.html "13.3.6 LOCK TABLES and UNLOCK TABLES Statements") to be released, as though you had executed [`UNLOCK TABLES`](https://dev.mysql.com/doc/refman/8.0/en/lock-tables.html "13.3.6 LOCK TABLES and UNLOCK TABLES Statements"). Beginning a transaction does not release a global read lock acquired with [`FLUSH TABLES WITH READ LOCK`](https://dev.mysql.com/doc/refman/8.0/en/flush.html#flush-tables-with-read-lock).

## Preventions/Solutions to Deadlock

- Create indexes for the columns that are used as filters in the SQL queries.
- Break large batch inserts into smaller ones to shorten lock occupancy.
- When multiple rows are updated in different transactions, make sure they are updated in the same order.

- Our idea proposed to prevent deadlock:
  
  - Container table exaple:

    ```sql
    CREATE TABLE container (
      id BIGINT UNSIGNED AUTO_INCREMENT,
      clientId BIGINT UNSIGNED NOT NULL,
      code VARCHAR(50) NOT NULL,
      --Default Version 0
      version INT 0,
      description TEXT NULL,
      width INT NULL,
      length INT NULL,
      height INT NULL,
      maxWeight INT NULL,
      PRIMARY KEY (id),
      UNIQUE KEY (code),
    ) ENGINE=INNODB;
    ```

  - Example data for the table:

    | id | clientId | code | version | description | width | length | height | maxWeight |
    |----|----------|------|---------|-------------|-------|--------|--------|-----------|
    | 1 | 1 | CONT01 | 0 | TEST | 10 | 20 | 10 | 200 |
    | 2 | 1 | CONT02 | 0 | TEST | 10 | 20 | 10 | 200 |
    | 3 | 2 | CONT03 | 0 | TEST | 10 | 20 | 10 | 200 |

  - **Client1** access to the containers with `[get]/containers` and then updates one container:

    ```sql
    BEGIN; -- CLIENT1 gets the containers data at the same time that CLIENT2.
    SELECT * FROM container
    WHERE id=2;
    ```

    ```sql
    BEGIN;
    UPDATE container 
    SET code='CONT01-A', version=version+1 
    WHERE id=2 AND version=0;
    -- Now the version will be 1
    ```

  - **Client2** access to the containers with `[get]/containers` and then updates one container after the **Client1** updated the container:

    ```sql
    BEGIN; -- CLIENT2 gets the containers data at the same time that CLIENT1.
    SELECT * FROM container
    WHERE id=2;
    ```

    ```sql
    BEGIN; -- CLIENT2 has to wait till CLIENT1 updates the CONTAINER
    UPDATE container 
    SET code='CONT01-B', version=version+1 
    WHERE id=2 AND version=1;
    -- Now the version will be 2
    ```
