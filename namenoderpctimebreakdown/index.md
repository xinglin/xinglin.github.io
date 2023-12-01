# HDFS Namenode RPC Request Execution Time Breakdown


```
 -----                                                enqueueTime  queueingTime      Processing/ResponseTime/HandlerTime 
|50090| <- Listener --> pendingConnections <- Reader1 -----------> CallQueue <----- handler (processes and sends response)
 -----              \-> pendingConnections <- Reader2
```

A main listener thread is accepting new connections from clients and
put connections into pendingConnections queue of a Reader thread. 
A Reader thread detects any ready connection, reads the request and puts
the call into CallQueue. This put() operation is blocking and is accounted as
enqueueTime. The time a call stays in CallQueue is queueingTime.
When a handler is available, it will pick a call from CallQueue and process the
call, from which we can derive the processing/response/handlerTime.

Hadoop supports adding more listener threads by specifying the auxiliary listener ports `dfs.namenode.rpc-address.auxiliary-ports`.
