# Hello Dynamo
Here are examples and tutorials to understand the position and features of Dynamo

## Single Node Example
- Example [repo](https://github.com/ai-dynamo/dynamo/blob/main/examples/hello_world/hello_world.py)
- Frontend/middle/backend monolith application
```
Users/Clients (HTTP)
      │
      ▼
┌─────────────┐
│  Frontend   │  HTTP API endpoint (/generate)
└─────────────┘
      │ dynamo/runtime
      ▼
┌─────────────┐
│   Middle    │
└─────────────┘
      │ dynamo/runtime
      ▼
┌─────────────┐
│  Backend    │
└─────────────┘
```
## Multi Nodes Example
- Example [repo](https://github.com/ai-dynamo/dynamo/pull/624)
- Frontend/middle on one node and deploy backend workers to multi nodes
- random/round robin routing to worker nodes ( No workload awareness)

```
Users/Clients (HTTP)
      │
      ▼
┌─────────────────────┐
│  Frontend (node 1)  │  HTTP API endpoint (/generate)
└─────────────────────┘
      │ dynamo/runtime
      ▼
┌─────────────────────┐
│  Processor (node 1) │  ─────────────────
└─────────────────────┘   routing         │
      │ dynamo/runtime                    │ dynamo/runtime
      ▼                                   ▼
┌─────────────────────┐        ┌─────────────────────┐
│  Worker_1  (node 2) │        │  Worker_2  (node 3) │
└─────────────────────┘        └─────────────────────┘
```
## Multi Nodes Disagg skeleton 
- Example [repo](https://github.com/ai-dynamo/dynamo/tree/main/examples/hello_world/disagg_skeleton)
- A simplified **disaggregated serving architecture** used in LLM examples 
- It removes the LLM related inference code and focuses on how Dynamo handles kv routing, task queue and metadata communication between prefill and decode workers.
- Refer to [disaggregated](https://github.com/ai-dynamo/dynamo/blob/main/docs/disagg_serving.md) and [backend](https://github.com/ai-dynamo/dynamo/blob/main/docs/backend.md) documentation for more details
```
                                                 +----------------+
                                                 | prefill worker |-------+
                                                 |                |       |
                                                 +----------------+       | pull
                                                                          v
+------+      +-----------+      +------------------+    push     +---------------+
| HTTP |----->| processor |----->|  decode/monolith |------------>| prefill queue |
|      |<-----|           |<-----|      worker      |             |               |
+------+      +-----------+      +------------------+             +---------------+
                  |    ^
       query best |    | return
           worker |    | worker_id
                  |    |         +------------------+
                  |    +---------|      router      |
                  +------------->|                  |
                                 +------------------+

```

