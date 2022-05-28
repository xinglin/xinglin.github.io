# Postgres Handling Transaction ID Wrap Around


**NOTE**: OpenGauss may fix this problem. 

Three special values are reserved as special transaction IDs,
including invalid, bootstrap and frozen transactions. 
The rest values are used in a circular fashion. 

Once a row is older enough (controlled by *vacuum_freeze_min_age*, an 
integer value measuring the number of transactions),
the vaccum process can mark its TID as FrozenTransactionId.
FrozenTransactionId is smaller than any normal transaction ID
and thus these frozen rows are visible for all current
and future transactions.  

NOTE:  
The requirement of a periodical vaccum process to set rows as frozen 
(set TID to FrozenTransactionId), in order to handle TID wraparound 
problem does not seem optimal/elegant. Is there any better way to handle it?

```
/* ----------------
 *		Special transaction ID values
 *
 * BootstrapTransactionId is the XID for "bootstrap" operations, and
 * FrozenTransactionId is used for very old tuples.  Both should
 * always be considered valid.
 *
 * FirstNormalTransactionId is the first "normal" transaction id.
 * Note: if you need to change it, you must change pg_class.h as well.
 * ----------------
 */
#define InvalidTransactionId		((TransactionId) 0)
#define BootstrapTransactionId		((TransactionId) 1)
#define FrozenTransactionId			((TransactionId) 2)
#define FirstNormalTransactionId	((TransactionId) 3)
#define MaxTransactionId			((TransactionId) 0xFFFFFFFF)

#define TransactionIdIsValid(xid)		((xid) != InvalidTransactionId)
#define TransactionIdIsNormal(xid)		((xid) >= FirstNormalTransactionId)

/* advance a transaction ID variable, handling wraparound correctly */
#define TransactionIdAdvance(dest)	\
	do { \
		(dest)++; \
		if ((dest) < FirstNormalTransactionId) \
			(dest) = FirstNormalTransactionId; \
	} while(0)


/* back up a transaction ID variable, handling wraparound correctly */
#define TransactionIdRetreat(dest)	\
	do { \
		(dest)--; \
	} while ((dest) < FirstNormalTransactionId)
```
