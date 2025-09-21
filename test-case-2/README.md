# Test Case 2: Preemption Inside Same Queue

## Overview
This test case validates preemption within the same queue when multiple applications are competing for resources. It tests how fair share preemption works when applications within the same queue need to preempt each other.

## Test Scenario
```
root: 1000 CPU
├── root.GroupA: guarantee=500 CPU
│   ├── root.GroupA.ChildX: guarantee=100 CPU
│   └── root.GroupA.ChildY: guarantee=100 CPU
└── root.GroupB: guarantee=500 CPU
```

## Expected Behavior

### Phase 1: Initial Allocation (Same as Case 1)
1. **App1 → ChildX**: Request 600 CPU → Gets 600 CPU
2. **App2 → ChildY**: Request 600 CPU → Gets 400 CPU
   - **Preemption**: ChildX is overusing by 100 CPU (600 - 500 = 100)
   - **Result**: ChildX gets 500 CPU, ChildY gets 500 CPU

### Phase 2: Additional Application in Same Queue
3. **App3 → ChildY**: Request 600 CPU → Gets 0 CPU initially
   - **Fair share calculation**: ChildY fair share = 500 CPU / 2 = 250 CPU per app
   - **Current usage**: App2 in ChildY is using 500 CPU (over fair share by 250 CPU)
   - **Preemption**: App3 preempts 200 CPU from App2 (each container consumes 100 CPU in our setup)
   - **Result**: App2 gets 300 CPU, App3 gets 200 CPU

## Key Validation Points
- Fair share calculation works within the same queue: 500 / 2 = 250 CPU per app
- Preemption can occur between applications in the same queue
- Each application gets its fair share within the queue
- Resource preemption respects container-level resource requirements (100 CPU per container)
- Final allocation: App2 gets 300 CPU, App3 gets 200 CPU in ChildY

## Files
- `queues.yaml`: Queue configuration with GroupA containing ChildX and ChildY
- `job-app1.yaml`: Job for ChildX that requests 600 CPU
- `job-app2.yaml`: Job for ChildY that requests 600 CPU
- `job-app3.yaml`: Additional job for ChildY that requests 600 CPU
