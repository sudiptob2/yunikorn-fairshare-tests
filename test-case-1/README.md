# Test Case 1: Basic Fairshare Preemption

## Overview
This test case validates basic fair share preemption between sibling queues. It tests the fundamental fair share calculation and preemption mechanism when one queue exceeds its fair share allocation.

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

### Phase 2: Fair Share Calculation and Preemption
- **Fair share calculation**: total_allocation_of_parent / active_child = 1000 / 2 = 500 CPU per child
- **ChildX**: Currently using 600 CPU (over fair share by 100 CPU)
- **ChildY**: Currently using 400 CPU (under fair share by 100 CPU)
- **Preemption**: ChildX is overusing by 100 CPU, so 100 CPU is preempted from ChildX
- **Result**: ChildX gets 500 CPU, ChildY gets 500 CPU (100 CPU transferred from ChildX to ChildY)

## Key Validation Points
- Fair share calculation works correctly: 1000 / 2 = 500 CPU per child
- Preemption identifies the overusing queue (ChildX)
- Resource transfer occurs from overusing queue to underusing queue
- Final allocation respects fair share limits
- Both children end up with equal allocations (500 CPU each)

## Files
- `queues.yaml`: Queue configuration with GroupA containing ChildX and ChildY
- `job-app1.yaml`: Job for ChildX that requests 600 CPU
- `job-app2.yaml`: Job for ChildY that requests 600 CPU
