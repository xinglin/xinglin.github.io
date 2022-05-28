# Crash Recovery


The following content is selected from the Database Management Systems book, chapter 18. 

# ARIES
> Three main principles lie behind the ARIES recovery algorithm:
> 1. Write-Ahead Logging: Any change to a database object is first recorded in the log; the record in the log must be written to stable storage before the change to the database object is written to disk.
> 2. Repeating History During Redo: On restart following a crash, ARIES retraces all actions of the DBMS before the crash and brings the system back to the exact state that it was in at the time of the crash. Then, it undoes the actions of transactions still active at the time of the crash. (effectively aborting them).
> 3. Logging Changes During Undo: Changes mada to the database while undoing a transaction are logged to ensure such an action is not repeated in the event of repeated (failures causing) restarts.

> Every log record is given a unique id called the log sequence number (LSN).
> For recovery purposes, every page in the database contains the LSN of the most recent log record that describes a change to this page. This LSN is called the pageLSN.
> Every log record has certain fields: prevLSN, transID, and type. The set of all log records for a given transaction is maintained as a linked list going back in time, using the prevLSN field; this list must be updated whenever a log record is added. The transID field is the id of the transaction generating the log record, and the type field obviously indicates the type of the log record.

* Update log record format
```
struct CommonHeader {
    long prevLSN;
    long transID;
    byte  type;
};

struct UpdateRec {
    CommonHeader head;
    long pageID
    int length
    int offset
    void* before-image;
    void* after-image;
};
```
> The before-image is the value of the changed bytes before the change; the after-image is the value after the change. A redo-only update log record contains just the after-image; similarly an undo-only update record contains just the before-image.



* Actions need to be logged
>   Updating a Page: After modifying the page, an update type record (described later in this section) is appended to the log tail. The pageLSN of the page is then set to the LSN of the update log record. (The page must be pinned in the buffer pool while these actions are carried out.)
>   Commit: when a transaction decides to commit, it force-writes a commit type log record containing the transaction id. That is, the log record is appended to the log, and the log tail is written to stable storage, up to and including the commit record. The transaction is considered to have committed at the instant that its commit log record is written to stable storage. (Some additional steps must be taken, e.g., removing the transaction's entry in the transaction table; these follow the writing of the commit log record.)
>   Abort: When a transaction is aborted, an abort type log record containing the transaction id is appended to the log, and Undo is initiated for this transaction (Section 18.6.3).
>   End: As noted above, when a transaction is aborted or committed, some additional actions must be taken beyond writing the abort or commit log record. After all these additional steps are completed, an end type log record containing the transaction id is appended to the log.
>   Undoing an update: When a transaction is rolled back (because the transaction is aborted, or during recovery frorn a crash), its updates are undone. When the action described by an update log record is undone, a compensation log record, or CLR, is written.

> In addition to the log, the following two tables contain important recovery-related infornlation:
>   Transaction Table: This table contains one entry for each active transaction. The entry contains (among other things) the transaction id, the status, and a field called lastLSN, which is the LSN of the most recent log record for this transaction. The status of a transaction can be that it is in progress, corunlitted, or aborted. (In the latter two cases, the transaction will be removed from the table once certain 'clean up' steps are completed.)
>   Dirty page table: This table contains one entry for each dirty page in the buffer pool, that is, each page with changes not yet reflected on disk. The entry contains a field recLSN, which is the LSN of the first log record that caused the page to becorne dirty. Note that this LSN identifies the earliest log record that might have to be redone for this page during restart from a crash.

## Write-ahead log procotol
>   Before writing a page to disk, every update log record that describes a change to this page must be forced to stable storage. This is accomplished by forcing all log records up to and including the one with LSN equal to the pageLSN to stable storage before writing the page to disk.
>   The definition of a committed transaction is effectively 'a transaction all of whose log records including a commit record have been written to stable storage'.
