# Test Case: Opt-Out Preemption

## Overview
This test case validates the preemption opt-out mechanism in YuniKorn. It demonstrates how pods can opt out of preemption using the `yunikorn.apache.org/allow-preemption: "false"` annotation, preventing them from being preempted even when they exceed their fair share allocation.

## Test Scenario
```
root: 1000 CPU
├── root.GroupA: guarantee=500 CPU
│   ├── root.GroupA.ChildX: guarantee=100 CPU
│   └── root.GroupA.ChildY: guarantee=100 CPU
└── root.GroupB: guarantee=500 CPU
```

## Expected Behavior

### Phase 1: Initial Allocation
1. **App1 → ChildX**: Request 600 CPU → Gets 600 CPU (initially available)
2. **App2 → ChildY**: Request 600 CPU → Gets 400 CPU (limited by available resources)

### Phase 2: Preemption Attempt
- **Fair share calculation**: total_allocation_of_parent / active_child = 1000 / 2 = 500 CPU per child
- **ChildX**: Currently using 600 CPU (over fair share by 100 CPU)
- **ChildY**: Currently using 400 CPU (under fair share by 100 CPU)
- **Preemption attempt**: ChildY tries to preempt 100 CPU from ChildX
- **Opt-out behavior**: All pods in ChildX have `yunikorn.apache.org/allow-preemption: "false"` annotation, so they cannot be preempted
- **Result**: ChildX keeps 600 CPU, ChildY remains at 400 CPU (no preemption occurs)

## Key Validation Points
- Fair share calculation works correctly: 1000 / 2 = 500 CPU per child
- Preemption identifies the overusing queue (ChildX)
- Opt-out annotation prevents preemption of pods in ChildX
- No resource transfer occurs when pods opt out of preemption
- Final allocation reflects the opt-out behavior: ChildX 600 CPU, ChildY 400 CPU

## Files
- `queues.yaml`: Queue configuration with GroupA containing ChildX and ChildY
- `job-app1.yaml`: Job for ChildX that requests 600 CPU (with preemption opt-out annotation)
- `job-app2.yaml`: Job for ChildY that requests 600 CPU (without opt-out annotation)
