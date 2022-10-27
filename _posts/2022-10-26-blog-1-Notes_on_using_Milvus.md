---
title: 'Notes on using Milvus'
date: 2022-10-26
permalink: /posts/2022/10/blog-1-Notes_on_using_Milvus/
tags:
  - Database
---

Notes on using Milvus v2.1.x

### 0. Installing Milvus v2 via Docker: APT vs Docker-compose

If you are using Docker to install Milvus v2, it is better to choose the docker-compose solution rather than using APT inside docker container. This is because the latter way requires to start Milvus with `systemctl `, which normally does not work in a docker container. 

### 1. Internal error when inserting too many vectors

If you try to insert a large number of vectors into Milvus, do not insert all of them in one call of `insert` method, otherwise you may encounter this error:

```
<_InactiveRpcError of RPC that terminated with:
	status = StatusCode.INTERNAL
	details = "Exception serializing request!"
	debug_error_string = "None"
>
```

This is caused by the message size limit of Protobuf. Milvus uses Protobuf for process communication, but Protobuf has a hard limit of 2GB on the message size, because many implementations use 32-bit signed arithmetic. Therefore, you have to make sure the size of the data inserted each time is less than 2GB. A better way is to split the large amount of vectors into multiple batches, and only insert one batch each time, such that each batch has a size smaller than 2GB. 

For more details, see these links:

* [Internal Error when inserting too many vectors · Issue #1438 · milvus-io/milvus (github.com)](https://github.com/milvus-io/milvus/issues/1438)
* [protocol buffers - google protobuf maximum size - Stack Overflow](https://stackoverflow.com/questions/34128872/google-protobuf-maximum-size)

### 2. grpc error: received message larger than max

Similar to the case of inserting data, when querying the data, if the returned results are too large, you may see this error:

```
err: rpc error: code = ResourceExhausted desc = grpc: received message larger than max
```

This is because the grpc also has size limit on the message. So it is better to ask the system to return less data, like returning vector IDs instead of the vectors themselves.

See these links for details:

* [[Bug\]: err: rpc error: code = ResourceExhausted desc = grpc: received message larger than max (215849469 vs. 104857600) · Issue #18558 · milvus-io/milvus (github.com)](https://github.com/milvus-io/milvus/issues/18558)

