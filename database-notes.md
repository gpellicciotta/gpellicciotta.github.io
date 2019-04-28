# Database Notes

Notes on DB related topics.

## Database Schema Evolution without Downtime

### External References
- [Zero-Downtime SQL Database SchemaEvolution for Continuous Deployment](https://repository.tudelft.nl/assets/uuid:af89f8ba-fc34-4084-b479-154be397718f/thesis.pdf])

### Situation:
Imagine an app with following architecture:
 * many clients that connect concurrently ...
 * via loadbalancers to ...
 * 2 application servers that run the main business logic, which in turn use...
 * 2 database servers (one active, one passive for DR) for persistent storage

**Question #1**: can we evolve the database schema (e.g. add column, rename tables, ...) with zero downtime?

Before answering this question, lets first answer a simpler **question #2**: can we evolve the code base with zero downtime?

Answer to **question #2**:  
Yes, as follows:
 1. Configure load balancers to bring all new traffic to app. server 2
 2. Wait until all existing sessions on app. server 1 have ended
 3. Upgrade code on app. server 1
 4. Route all new sessions to app. server 1
 5. Wait until all existing sessions on app. server 2 have ended *
 6. Upgrade code on app. server 2
 7. Load balance all traffic again over two app. servers
  
Assumptions for answer to **question #2**:
* Online sessions have a limited timespan of a couple of hours to a day
* The code change does not prevent old sessions (with old code) and new sessions (with new code) to run concurrently. Most typically this issue only happens in combination with new/old code relying on specific database structures or data being available, which is question #1's problem statement

**Question #3**: could we evolve the code base with zero downtime if there were only 1 app. server and no load balancing?  
Answer to **question #3**:
 * Yes, but a provision would have to be made from the beginning that enable two different code bases running on the same server and to 'route' new sessions to either code base
 * For example: new sessions are established by starting a script that reads a config file to determine which code to execute

So... back to **question #1**: can we do something similar with the database?   
Not so easy, although there are more restricted scenario's thinkable where approx. the same process could be employed, as follows (and we're assuming here that a massive schema change also always means an accompanying code change)
 1. Configure load balancers to bring all new traffic to app. server 1 only
 2. Wait until all existing sessions on app. server 2 have ended [1]
 3. Configure app. server 2 to only allow read-only operations [2] and to work with db2 directly (or do we need a db3 for that, just in case there is a disaster in the middle of this upgrade procedure or because of technical restrictions?)
 4. Configure load balancers to bring all new traffic to app. server 2 only
 5. Wait until all existing sessions on app. server 1 have ended [1] (at this point there are only read-only sessions on app server 2 + db2)
 6. Ensure db2 is in sync with db1 (who has been getting updates until this point: should be in sync without explicit action?) and decouple the two databases
 7. Upgrade code on app. server 1
 8. Upgrade db 1 also
 9. Route all new sessions to app. server 1
10. Wait until all existing sessions on app. server 2 have ended *
11. Upgrade code on app. server 2 + ensure it uses db1 again
12. Load balance all traffic again over two app. servers again

[1]: The big assumption here is that a) it is possible to run in read-only mode: both technically and organizationally.
[2]: Another (reasonable?) assumption is that db1 and db2 can be kept in sync, decoupled and brought in sync again (despite one db being structurally changed) with zero downtime.

What if we cannot make this 'read-only' assumption, if there are business requirements that force us to have always update scenario's?

Then... things are more complicated and the code base will have to get more complicated to enable this:
 1. Configure load balancers to bring all new traffic to app. server 1 only
 2. Wait until all existing sessions on app. server 2 have ended [1]
 3. Configure app. server 2 to:
    1. Work exclusively with db3, a copy of db1: so the existing code on app server 2 can remain using its existing database access code
    2. Record all update commands (insert/update/delete) in a new db4 that will act as a kind of FIFO command buffer [3]
 4. Configure load balancers to bring all new traffic to app. server 2 only
 5. Wait until all existing sessions on app. server 1 have ended [1] (at this point db1/2 are not being used)
 6. Upgrade code on app. server 1 + upgrade db 1 also
 7. Start applying the datatabase updates recording in db4 to db1/2 (possibly re-interpreting the commands in function of the new database structure so the commands should preferably be stored not as raw SQL but as higher-level operations) until db1 is not more than x commands/time behind db3/4; this process should stop automatically once it is known that no new updates will get added (i.e. after step 9)
 8. Route all new sessions to app. server 1 (only once db1 is only a very limited amount of updates behind on db3)
 9. Wait until all existing sessions on app. server 2 have ended [1] (+ indicate this as special 'END of UPDATES' command in db4)
10. Upgrade code on app. server 2 + ensure it uses db1 again
11. Load balance all traffic again over two app. servers again
12. Once all updates from db4 are applied and the update process finished: remove db4

[3]: The big assumptions in this scenario are:
1. Updates can be recorded in a new FIFO queue (complexity of this depends on application code architecture but in principle is always possible)
2. DB3 can be created and brought in sync with db1 with downtime for db1
3. Once app server 1+db 1 are used again, db1 will be behind on db3 for some period of time (until the database update process has finished) and this 'being' behind has no serious consequence (e.g. db1 update succeeds (INSERT new user id) but would have failed in all db4 updates were already applied)
4. Online sessions have a limited timespan of a couple of hours to a day
5. The code change does not prevent old sessions (with old code) and new sessions (with new code) to run concurrently (but I cannot think of an example where this would be an issue)

In order to enable the above scenario, the application's architecture from the start should enable:
* Recording update requests in a high-level fashion
* Not critically relying on database updates being applied instantaneously (but: example where this reliance exists?) or even in order
* For mitigating the updates must absolutely be applied in order issue, the application's architecture could be setup in a way to always do updates asynchronously:
* Updates are always first put in a FIFO 'command queue'
* There is an async process that applies the updates

When this architecture is in place, db1's 'command queue' can get filled with db4 + new commands from app. server 1 sessions are also added there. As long as all commands have a timestamp or sequence that is granular enough (and shared by all systems involved) to put commands into an absolute sequence, this will prevent updates from being applied 'out of order'.

But... is it realistic to only be able to do async updates? It seems to conflict with the idea of transactions:
* Q: How to apply (or rollback) multiple related updates if they are not updated together?  
  A: Still treat them together as one high-level update operation

* Q: How to react properly to error situations since they won't be immediate?  
  A: Wait somehow until the async update got eventually applied and the error code is known... 

* Q: But this could be a long wait, leading to bad UX  
  A: Assuming the async. update process is most of the time pretty fast and not much behind, a strategy could be coded to wait e.g. 300ms for confirmation about success/failure and if no response yet: indicate to the user that the operation takes longer than usual, possibly providing a 'reference link' to check later.  
  A: Another option could be to store some kind of max.time.till.completion value with the high-level update operation and the async update process checking whether the value has not expired yet: if it has, don't even execute the operation and mark it as failed. If it hasn't expired yet, execute but after succesful exection check again whether the max. time was not surpassed and only then COMMIT and mark as succeeded. Code should then wait a little longer than the configured max. time to take network delay etc. into account but is mostly guaranteed to know at a certain time whether it was succesful or not

There might be another possibility to do this kind of upgrade by:
  1. Upgrading code + databaschema incrementally on app/db server 1 exclusively
  2. Database schema changes can be done by creating 'shadow tables' that get updated with new data by having new triggers on the original tables + having a background process that pumps over all old/existing data
  3. Once all shadow tables are fully populated and kept in sync: change the code to use them directly instead of the old tables
  4. Then drop the now unused old tables
  
This approach seems to have many advantages:
* brings the burden of recording + shipping-while-transforming update operations to a new database schema down by using basic capabilities of the underlying database, augmented by additional (more complex) code. 
* there is no need to async updates with this approach.
* the data sync process is limited to applying within a single database, which should be a lot simpler to get right, and also a lot faster

Assumptions and possible issues:
* The database to be upgraded might, depending the magnitude of the schema change, have to host twice the number of data, at least until the full upgrade process has completed: this might have repercussions on performance too, together with the additional sync processing that is happening constantly
* Switching from using the old data tables to the new must be done in a way that doesn't interrupt operations... assuming we cannot have a 'read-only' period (there are constant updates) and it is also not possible to switch all code at once (since at each moment in time old code will be running): I can only see this working by introducing a kind of reverse sync process: from updates updating data in the new tables to the old tables (and avoiding recursion!). That way, code can get upgraded in a roundabout way.
  * This is itself assumes that sync + reverse sync processes can be introduced without downtime (e.g. with triggers only if creating/changing/dropping them can be done during normal operations; alternative is in code entirely)


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
  
