## Big Picture: What Is a Controller?

A controller is:
An event-driven, level-based reconciliation loop that continuously drives actual state toward desired state.
Core pipeline:

```
Informer → WorkQueue → Workers → Reconcile()
```

### 1️⃣ Informer
What it is

An Informer is a cached watcher for Kubernetes resources.
It does:

- List existing objects
- Watch for changes
- Maintain local cache
- Emit events (Add / Update / Delete)

Built using:

client-go/tools/cache

#### Why Informers Exist

- Without informers Every reconcile would hit API server
- Massive load
- No caching
- No event-driven triggers

##### Informer gives:

- Shared cache
- Event stream
- Efficient API usage
- Important Internals

##### Informer uses:

- Reflector → watches API server
- DeltaFIFO → stores change events
- Local Store (thread-safe cache)

### 2️⃣ WorkQueue
What it is

Thread-safe queue from:

client-go/util/workqueue


It stores:

- namespace/name (reconcile.Request)
- NOT full objects.

#### Why WorkQueue Exists

It provides:

- Decoupling between events and processing
- Rate limiting
- Retry with exponential backoff
- Deduplication
- Safe concurrency

#### Important Behavior

- Same key won’t be queued multiple times simultaneously
- Failed reconcile automatically requeued (if error returned)
- Supports delayed requeue

### 3️⃣ Workers

Workers are:

- Goroutines that continuously pull from the workqueue.

Internally:

```go
for {
    item := queue.Get()
    Reconcile(item)
    queue.Done(item)
}
```

- Each worker processes one item at a time.

### 4️⃣ Goroutines

Controller-runtime spawns:

👉 MaxConcurrentReconciles: number of worker goroutines.

Example:

```
WithOptions(controller.Options{
    MaxConcurrentReconciles: 5,
})
```
This means:
- 5 concurrent reconcile executions max
- One shared queue
- Parallel processing enabled

### 5️⃣ Parallel Reconciliation

Parallelism happens when:
- Queue has multiple items
- MaxConcurrentReconciles > 1

Example:

```
Queue:
A, B, C, D

Workers:
W1 → A
W2 → B
W3 → C
Parallel.
```

Important Guarantee

- Same object key will NOT be processed concurrently.
- Deduplication prevents:

Reconcile(A)
Reconcile(A)

running at same time.

### 6️⃣ Reconcile Function
Reconcile is Stateless, idempotent, level-based function.
It must Read current state and Compare with desired state, Make corrections
Be safe to run multiple times

Never assume order.
Never assume single execution.
Never depend on previous run.

### 7️⃣ Desired State vs Actual State

- Desired State: Defined in CR .spec
- Actual State:What exists in cluster (Pods, CM, etc.)
- Reconcile always computes:diff(desired, actual)
- Then converges system.
- This is called:Level-based reconciliation (not step-based)

8️⃣ Retry & Backoff

- If Reconcile returns: return ctrl.Result{}, err
- Controller-runtime does: queue.AddRateLimited(key)

Which applies exponential backoff.

If you return:
return ctrl.Result{RequeueAfter: 30 * time.Second}, nil
It schedules delayed requeue.

9️⃣ Leader Election

Workqueue is in-memory.
If multiple pods run controller:

- Each would have its own queue
- Could process same object

So controller-runtime enables: LeaderElection: true

Only one pod becomes leader and runs controllers and Others standby.

Leader election uses: Lease object in cluster


### 🔟 Scaling Model

Two types of scaling:

#### Vertical (Inside One Pod)

```
Controlled by:
MaxConcurrentReconciles
```
More goroutines → more parallelism.

#### Horizontal (Multiple Pods)

Controlled by:

- Kubernetes Deployment replicas
- Leader Election But only one leader actively reconciles.

```Controller Execution Flow (Complete)
API Server
   ↓
Reflector (Watch)
   ↓
Informer Cache
   ↓
Event Handler
   ↓
queue.Add(key)
   ↓
Worker goroutines
   ↓
Reconcile(key)
   ↓
Update resources
   ↓
Update status
```

###  Why Queue Is Not Durable

Queue is in-memory.
If controller crashes:
- Queue lost
- On restart, informer relists all objects get re-enqueued
- Durability lives in: Kubernetes API server.


### Common Misconceptions

❌ Reconcile is triggered once
→ It can run anytime.

❌ Queue stores objects
→ It stores only keys.

❌ Multiple replicas scale controller
→ Only leader processes.

❌ Reconcile is step-based
→ It is level-based.

### One-Line Definitions (For Interviews)

```
Informer → Cached watcher of Kubernetes resources
WorkQueue → Thread-safe rate-limited queue of reconcile keys
Workers → Goroutines consuming queue
MaxConcurrentReconciles → Worker pool size
Leader Election → Ensures only one active controller instance
Reconcile → Idempotent state convergence function
```
