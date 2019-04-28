# Database Notes

Notes on DB related topics.

## Transaction Logs

From http://www.techrepublic.com/article/understanding-the-importance-of-transaction-logs-in-sql-server/:
- SQL Server keeps a buffer of all of the changes to data for performance reasons. It writes items to the transaction log immediately, but does not write changes to the data file immediately. A checkpoint is written to the transaction log to indicate that a particular transaction has been completely written from the buffer to the data file. When SQL Server is restarted, it looks for the most recent checkpoint in the transaction log and rolls forward all transactions that have occurred from that point forward since it is not guaranteed to have been written to the data file until a checkpoint is entered in the transaction log. This prevents transactions from being lost that were in the buffer but not yet written to the data file.

From http://dba.stackexchange.com/questions/75821/checkpoint-or-commit-writes-to-disk/75823:
- All fully logged data writes (changes) occur in the exactly following sequence:
  - The data page is latched exclusively
  - A log record describing the change is added to to log, in memory. New log record generates a new LSN, see What is an LSN: Log Sequence Number.
  - The data page is modified (both data record and last_update_lsn on the page). This is now modified ('dirty') page.
  - The data page latch is released nothing gets written to disk directly as the result of the update
- A COMMIT does the following
  - adds a new log record describing the COMMIT to the log, in memory
  - all log records not flushed to disk, up to and including the one generated above, are flushed (written to disk)
  - thread blocks waits until the OS reports the above write as durable (IO completes)
  - COMMIT statement (or DML statement with implicit commit) completes
- A CHECKPOINT does the following (simplified), see How do checkpoints work and what gets logged:
  - All dirty pages in memory are written to disk
  - For each dirty page, before starting to write to disk, the log up to and including the LSN that is the last_update_lsn on that page is flushed (written to disk). Note that flushin any LSN implies all previous LSNs are also flushed, so for the most dirty pages this is a no-op since it's own last_update is likely already flushed.
  - log record describing the checkpoint is written to the log and flushed
  - the database boot page is update with the LSN of the record generated above
  
