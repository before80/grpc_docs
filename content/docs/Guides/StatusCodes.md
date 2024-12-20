+++
title = "Status Codes"
date = 2024-11-19T10:19:42+08:00
weight = 210
type = "docs"
description = ""
isCJKLanguage = true
draft = false
+++

> 原文：[https://grpc.io/docs/guides/status-codes/](https://grpc.io/docs/guides/status-codes/)
>
> 收录该文档的时间：`2024-11-19T10:19:42+08:00`

# Status Codes

Explains the status codes used in gRPC.



### Overview

All RPCs will result in a `status` being returned to the client. A `status` object is composed of an integer code and a string error description. The server-side (or the gRPC library for library level errors) chooses the status it returns for a given RPC. Applications should only use values defined below.

When an error situation occurs, the gRPC library may produce a corresponding `status`. The library may do this either on the client- or the server-side. Only a subset of the pre-defined status codes are generated by the gRPC libraries. This allows applications to be sure that any other code it sees was actually returned by the application (although it is also possible for the server-side to return one of the codes generated by the gRPC libraries).

See the [Error handling]({{< ref "/docs/Guides/Errorhandling" >}}) user guide for how to use status codes.

### Full list of Status codes

gRPC uses a set of well defined status codes as part of the RPC API.

The following status codes are never generated by the library, only by user code:

- INVALID_ARGUMENT
- NOT_FOUND
- ALREADY_EXISTS
- FAILED_PRECONDITION
- ABORTED
- OUT_OF_RANGE
- DATA_LOSS

#### The full list of status codes

| Code                | Id   | Description                                                  |
| ------------------- | ---- | ------------------------------------------------------------ |
| OK                  | 0    | Not an error; returned on success.                           |
| CANCELLED           | 1    | The operation was cancelled, typically by the caller.        |
| UNKNOWN             | 2    | Unknown error. For example, this error may be returned when a `Status` value received from another address space belongs to an error space that is not known in this address space. Also errors raised by APIs that do not return enough error information may be converted to this error. |
| INVALID_ARGUMENT    | 3    | The client specified an invalid argument. Note that this differs from `FAILED_PRECONDITION`. `INVALID_ARGUMENT` indicates arguments that are problematic regardless of the state of the system (e.g., a malformed file name). |
| DEADLINE_EXCEEDED   | 4    | The deadline expired before the operation could complete. For operations that change the state of the system, this error may be returned even if the operation has completed successfully. For example, a successful response from a server could have been delayed long enough for the deadline to expire. |
| NOT_FOUND           | 5    | Some requested entity (e.g., file or directory) was not found. Note to server developers: if a request is denied for an entire class of users, such as gradual feature rollout or undocumented allowlist, `NOT_FOUND` may be used. If a request is denied for some users within a class of users, such as user-based access control, `PERMISSION_DENIED` must be used. |
| ALREADY_EXISTS      | 6    | The entity that a client attempted to create (e.g., file or directory) already exists. |
| PERMISSION_DENIED   | 7    | The caller does not have permission to execute the specified operation. `PERMISSION_DENIED` must not be used for rejections caused by exhausting some resource (use `RESOURCE_EXHAUSTED` instead for those errors). `PERMISSION_DENIED` must not be used if the caller can not be identified (use `UNAUTHENTICATED` instead for those errors). This error code does not imply the request is valid or the requested entity exists or satisfies other pre-conditions. |
| RESOURCE_EXHAUSTED  | 8    | Some resource has been exhausted, perhaps a per-user quota, or perhaps the entire file system is out of space. |
| FAILED_PRECONDITION | 9    | The operation was rejected because the system is not in a state required for the operation’s execution. For example, the directory to be deleted is non-empty, an rmdir operation is applied to a non-directory, etc. Service implementors can use the following guidelines to decide between `FAILED_PRECONDITION`, `ABORTED`, and `UNAVAILABLE`: (a) Use `UNAVAILABLE` if the client can retry just the failing call. (b) Use `ABORTED` if the client should retry at a higher level (e.g., when a client-specified test-and-set fails, indicating the client should restart a read-modify-write sequence). (c) Use `FAILED_PRECONDITION` if the client should not retry until the system state has been explicitly fixed. E.g., if an “rmdir” fails because the directory is non-empty, `FAILED_PRECONDITION` should be returned since the client should not retry unless the files are deleted from the directory. |
| ABORTED             | 10   | The operation was aborted, typically due to a concurrency issue such as a sequencer check failure or transaction abort. See the guidelines above for deciding between `FAILED_PRECONDITION`, `ABORTED`, and `UNAVAILABLE`. |
| OUT_OF_RANGE        | 11   | The operation was attempted past the valid range. E.g., seeking or reading past end-of-file. Unlike `INVALID_ARGUMENT`, this error indicates a problem that may be fixed if the system state changes. For example, a 32-bit file system will generate `INVALID_ARGUMENT` if asked to read at an offset that is not in the range [0,2^32-1], but it will generate `OUT_OF_RANGE` if asked to read from an offset past the current file size. There is a fair bit of overlap between `FAILED_PRECONDITION` and `OUT_OF_RANGE`. We recommend using `OUT_OF_RANGE` (the more specific error) when it applies so that callers who are iterating through a space can easily look for an `OUT_OF_RANGE` error to detect when they are done. |
| UNIMPLEMENTED       | 12   | The operation is not implemented or is not supported/enabled in this service. |
| INTERNAL            | 13   | Internal errors. This means that some invariants expected by the underlying system have been broken. This error code is reserved for serious errors. |
| UNAVAILABLE         | 14   | The service is currently unavailable. This is most likely a transient condition, which can be corrected by retrying with a backoff. Note that it is not always safe to retry non-idempotent operations. |
| DATA_LOSS           | 15   | Unrecoverable data loss or corruption.                       |
| UNAUTHENTICATED     | 16   | The request does not have valid authentication credentials for the operation. |
