# vLLM backend Dynamo Deployment

## Single Node Example
- Example [repo](https://github.com/ai-dynamo/dynamo/tree/main/examples/llm)
- It has four different scenarios: agg/agg_route/disagg/disagg_route
- All processors, routers, prefill/decode workers are deployed on same node
```
                                                 +----------------+
                                          +------| prefill worker |-------+
                                   notify |      |                |       |
                                 finished |      +----------------+       | pull
                                          v                               v
+------+      +-----------+      +------------------+    push     +---------------+
| HTTP |----->| processor |----->| decode/monolith  |------------>| prefill queue |
|      |<-----|           |<-----|      worker      |             |               |
+------+      +-----------+      +------------------+             +---------------+
                  |    ^                  |
       query best |    | return           | publish kv events
           worker |    | worker_id        v
                  |    |         +------------------+
                  |    +---------|     kv-router    |
                  +------------->|                  |
                                 +------------------+

```

## Multi Node Example
### 1. Single node sized model
- Document [link](https://github.com/ai-dynamo/dynamo/blob/main/examples/llm/multinode-examples.md)
- We can reuse the single node vLLM example with `deepseek-ai/DeepSeek-R1-Distill-Llama-8B`
- Deploy agg graph **without** prefill worker
```
# graphs/agg_router.py
#Frontend.link(Processor).link(Router).link(VllmWorker)
cd $DYNAMO_HOME/examples/llm
dynamo serve graphs.agg_router:Frontend -f ./configs/agg_router.yaml
```
- Similar to the Disagg Skeleton example, deploy prefill worker separetely. 
```
cd $DYNAMO_HOME/examples/llm
dynamo serve components.prefill_worker:PrefillWorker -f ./configs/disagg.yaml
```
### 2. Multi nodes sized model 
- Will deploy Deepseek R1 as an example