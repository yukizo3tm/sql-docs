---
title: "BEGIN TRANSACTION (Transact-SQL)"
description: Marks the starting point of an explicit, local transaction. Explicit transactions start with the BEGIN TRANSACTION statement and end with the COMMIT or ROLLBACK statement.
author: rwestMSFT
ms.author: randolphwest
ms.date: 05/10/2024
ms.service: sql
ms.subservice: t-sql
ms.topic: reference
f1_keywords:
  - "BEGIN_TRANSACTION_TSQL"
  - "TRANSACTION_TSQL"
  - "TRANSACTION"
  - "BEGIN TRANSACTION"
  - "BEGIN TRAN"
  - "BEGIN_TRAN_TSQL"
helpviewer_keywords:
  - "transaction logs [SQL Server], BEGIN TRANSACTION statement"
  - "marked transactions [SQL Server], BEGIN TRANSACTION statement"
  - "BEGIN TRANSACTION statement"
  - "local transactions [SQL Server]"
  - "marking transactions [SQL Server]"
  - "transactions [SQL Server], starting"
  - "transaction names [SQL Server]"
  - "starting point marked for transactions"
  - "starting transactions"
dev_langs:
  - "TSQL"
monikerRange: ">=aps-pdw-2016 || =azuresqldb-current || =azure-sqldw-latest || >=sql-server-2016 || >=sql-server-linux-2017 || =azuresqldb-mi-current || =fabric"
---
# BEGIN TRANSACTION (Transact-SQL)

[!INCLUDE [sql-asdb-asdbmi-asa-pdw-fabricdw](../../includes/applies-to-version/sql-asdb-asdbmi-asa-pdw-fabricdw.md)]

Marks the starting point of an explicit, local transaction. Explicit transactions start with the `BEGIN TRANSACTION` statement and end with the `COMMIT` or `ROLLBACK` statement.

:::image type="icon" source="../../includes/media/topic-link-icon.svg" border="false"::: [Transact-SQL syntax conventions](../../t-sql/language-elements/transact-sql-syntax-conventions-transact-sql.md)

## Syntax

Syntax for SQL Server, Azure SQL Database, and Azure SQL Managed Instance.

```syntaxsql
BEGIN { TRAN | TRANSACTION }
    [ { transaction_name | @tran_name_variable }
      [ WITH MARK [ 'description' ] ]
    ]
[ ; ]
```

Syntax for Synapse Data Warehouse in Microsoft Fabric, Azure Synapse Analytics and Analytics Platform System (PDW).

```syntaxsql
BEGIN { TRAN | TRANSACTION }
[ ; ]
```

## Arguments

#### *transaction_name*

**Applies to:** [!INCLUDE [sql2008-md](../../includes/sql2008-md.md)] and later versions, Azure SQL Database, and Azure SQL Managed Instance

The name assigned to the transaction. *transaction_name* must conform to the rules for identifiers, but identifiers longer than 32 characters aren't allowed. Use transaction names only on the outermost pair of nested `BEGIN...COMMIT` or `BEGIN...ROLLBACK` statements. *transaction_name* is always case sensitive, even when the instance of [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] isn't case sensitive.

#### *@tran_name_variable*

**Applies to:** [!INCLUDE [sql2008-md](../../includes/sql2008-md.md)] and later versions, Azure SQL Database, and Azure SQL Managed Instance

The name of a user-defined variable containing a valid transaction name. The variable must be declared with a **char**, **varchar**, **nchar**, or **nvarchar** data type. If more than 32 characters are passed to the variable, only the first 32 characters are used. The remaining characters are truncated.

#### WITH MARK [ '*description*' ]

**Applies to:** [!INCLUDE [sql2008-md](../../includes/sql2008-md.md)] and later versions, Azure SQL Database, and Azure SQL Managed Instance

Specifies that the transaction is marked in the log. *description* is a string that describes the mark. A *description* longer than 128 characters is truncated to 128 characters before being stored in the `msdb.dbo.logmarkhistory` table.

If `WITH MARK` is used, a transaction name must be specified. `WITH MARK` allows for restoring a transaction log to a named mark.

## Remarks

`BEGIN TRANSACTION` increments `@@TRANCOUNT` by `1`.

`BEGIN TRANSACTION` represents a point at which the data referenced by a connection is logically and physically consistent. If errors are encountered, all data modifications made after the `BEGIN TRANSACTION` can be rolled back to return the data to this known state of consistency. Each transaction lasts until either it completes without errors and `COMMIT TRANSACTION` is issued to make the modifications a permanent part of the database, or errors are encountered and all modifications are erased with a `ROLLBACK TRANSACTION` statement.

`BEGIN TRANSACTION` starts a local transaction for the connection issuing the statement. Depending on the current transaction isolation level settings, many resources acquired to support the [!INCLUDE [tsql](../../includes/tsql-md.md)] statements issued by the connection are locked by the transaction until it completes with either a `COMMIT TRANSACTION` or `ROLLBACK TRANSACTION` statement. Transactions left outstanding for long periods of time can prevent other users from accessing these locked resources, and also can prevent log truncation.

Although `BEGIN TRANSACTION` starts a local transaction, it isn't recorded in the transaction log until the application then performs an action that must be recorded in the log, such as executing an `INSERT`, `UPDATE`, or `DELETE` statement. An application can perform actions such as acquiring locks to protect the transaction isolation level of `SELECT` statements, but nothing is recorded in the log until the application performs a modification action.

Naming multiple transactions in a series of nested transactions with a transaction name has little effect on the transaction. Only the first (outermost) transaction name is registered with the system. A rollback to any other name (other than a valid savepoint name) generates an error. None of the statements executed before the rollback is, in fact, rolled back at the time this error occurs. The statements are rolled back only when the outer transaction is rolled back.

The local transaction started by the `BEGIN TRANSACTION` statement is escalated to a distributed transaction if the following actions are performed before the statement is committed or rolled back:

- An `INSERT`, `DELETE`, or `UPDATE` statement that references a remote table on a linked server is executed. The `INSERT`, `UPDATE`, or `DELETE` statement fails if the OLE DB provider used to access the linked server doesn't support the `ITransactionJoin` interface.

- A call is made to a remote stored procedure when the `REMOTE_PROC_TRANSACTIONS` option is set to `ON`.

The local copy of [!INCLUDE [ssNoVersion](../../includes/ssnoversion-md.md)] becomes the transaction controller and uses [!INCLUDE [msCoName](../../includes/msconame-md.md)] Distributed Transaction Coordinator (MS DTC) to manage the distributed transaction.

A transaction can be explicitly executed as a distributed transaction by using `BEGIN DISTRIBUTED TRANSACTION`. For more information, see [BEGIN DISTRIBUTED TRANSACTION](begin-distributed-transaction-transact-sql.md).

When `SET IMPLICIT_TRANSACTIONS` is set to `ON`, a `BEGIN TRANSACTION` statement creates two nested transactions. For more information, see [SET IMPLICIT_TRANSACTIONS](../statements/set-implicit-transactions-transact-sql.md).

## Marked transactions

The `WITH MARK` option causes the transaction name to be placed in the transaction log. When you restore a database to an earlier state, the marked transaction can be used in place of a date and time. For more information, see [Use Marked Transactions to Recover Related Databases Consistently](../../relational-databases/backup-restore/use-marked-transactions-to-recover-related-databases-consistently.md) and [RESTORE Statements](../statements/restore-statements-transact-sql.md).

Additionally, transaction log marks are necessary if you need to recover a set of related databases to a logically consistent state. Marks can be placed in the transaction logs of the related databases by a distributed transaction. Recovering the set of related databases to these marks results in a set of databases that are transactionally consistent. Placement of marks in related databases requires special procedures.

The mark is placed in the transaction log only if the database is updated by the marked transaction. Transactions that don't modify data aren't marked.

`BEGIN TRANSACTION <new_name> WITH MARK` can be nested within an already existing transaction that isn't marked. Upon doing so, `<new_name>` becomes the mark name for the transaction, despite the name that the transaction might already have been given. In the following example, `M2` is the name of the mark.

```sql
BEGIN TRAN T1;

UPDATE table1 ...;

BEGIN TRAN M2 WITH MARK;
UPDATE table2 ...;
SELECT * from table1;

COMMIT TRAN M2;

UPDATE table3 ...;

COMMIT TRAN T1;
```

When you nest transactions, you receive the following warning message if you try to mark a transaction that is already marked:

```output
Server: Msg 3920, Level 16, State 1, Line 3
WITH MARK option only applies to the first BEGIN TRAN WITH MARK.
The option is ignored.
```

## Permissions

Requires membership in the **public** role.

## Examples

[!INCLUDE [article-uses-adventureworks](../../includes/article-uses-adventureworks.md)]

### A. Use an explicit transaction

**Applies to:** [!INCLUDE [sql2008-md](../../includes/sql2008-md.md)] and later versions, Azure SQL Database, Azure SQL Managed Instance, [!INCLUDE [ssazuresynapse-md](../../includes/ssazuresynapse-md.md)], Analytics Platform System (PDW)

```sql
BEGIN TRANSACTION;
DELETE FROM HumanResources.JobCandidate
    WHERE JobCandidateID = 13;
COMMIT;
```

### B. Rolling back a transaction

**Applies to:** [!INCLUDE [sql2008-md](../../includes/sql2008-md.md)] and later versions, Azure SQL Database, Azure SQL Managed Instance, [!INCLUDE [ssazuresynapse-md](../../includes/ssazuresynapse-md.md)], Analytics Platform System (PDW)

The following example shows the effect of rolling back a transaction. In this example, the `ROLLBACK` statement rolls back the `INSERT` statement, but the created table still exists.

```sql
CREATE TABLE ValueTable (id INT);
BEGIN TRANSACTION;
    INSERT INTO ValueTable VALUES(1);
    INSERT INTO ValueTable VALUES(2);
ROLLBACK;
```

### C. Name a transaction

**Applies to:** [!INCLUDE [sql2008-md](../../includes/sql2008-md.md)] and later versions, Azure SQL Database, Azure SQL Managed Instance

The following example shows how to name a transaction.

```sql
DECLARE @TranName VARCHAR(20);
SELECT @TranName = 'MyTransaction';

BEGIN TRANSACTION @TranName;
USE AdventureWorks2022;
DELETE FROM AdventureWorks2022.HumanResources.JobCandidate
    WHERE JobCandidateID = 13;

COMMIT TRANSACTION @TranName;
GO
```

### D. Mark a transaction

**Applies to:** [!INCLUDE [sql2008-md](../../includes/sql2008-md.md)] and later versions, Azure SQL Database, Azure SQL Managed Instance

The following example shows how to mark a transaction. The transaction `CandidateDelete` is marked.

```sql
BEGIN TRANSACTION CandidateDelete
    WITH MARK N'Deleting a Job Candidate';
GO
USE AdventureWorks2022;
GO
DELETE FROM AdventureWorks2022.HumanResources.JobCandidate
    WHERE JobCandidateID = 13;
GO
COMMIT TRANSACTION CandidateDelete;
GO
```

## Related content

- [BEGIN DISTRIBUTED TRANSACTION (Transact-SQL)](begin-distributed-transaction-transact-sql.md)
- [COMMIT TRANSACTION (Transact-SQL)](commit-transaction-transact-sql.md)
- [COMMIT WORK (Transact-SQL)](commit-work-transact-sql.md)
- [ROLLBACK TRANSACTION (Transact-SQL)](rollback-transaction-transact-sql.md)
- [ROLLBACK WORK (Transact-SQL)](rollback-work-transact-sql.md)
- [SAVE TRANSACTION (Transact-SQL)](save-transaction-transact-sql.md)
