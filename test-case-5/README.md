# Test Case 5: Guarantee Gets Prioritized

## Overview
This test case validates fair share preemption when queue guarantees take precedence over equal fair share distribution.

```
root: 1000 CPU
├── groupA: no guarantee specified
│   ├── childX: guarantee=600 CPU (large guarantee)
│   └── childY: guarantee=100 CPU (small guarantee)
└── groupB: no guarantee specified
```

## Expected Behavior

### Phase 1: Initial Allocation
1. **App1 → childX**: Request 800m CPU → Gets 800m CPU (initially available)
2. **App2 → childY**: Request 600m CPU → Gets 200m CPU (remaining cluster capacity)

### Phase 2: Fair Share Calculation and Preemption (Triggered by Demand)
3. **Fair share calculation**: total_allocation_of_parent / active_child = 1000 / 2 = 500m CPU per child
   - **ChildX guarantee**: 600m CPU > base fair share (500m CPU)
   - **Final fair share**: ChildX gets max(500m, 600m) = 600m CPU (guarantee takes precedence)
   - **ChildY fair share**: 1000m - 600m = 400m CPU (remaining after ChildX guarantee)
   - **Current state**: ChildX has 800m CPU, ChildY has 200m CPU
   - **Preemption needed**: ChildX is overusing by 200m CPU (800 - 600 = 200m over fair share)
   - **Preemption**: 200m CPU is preempted from ChildX to give ChildY its fair share
   - **Result**: ChildX gets 600m CPU, ChildY gets 400m CPU

### Phase 3: Intra-Queue Preemption
4. **App3 → childY**: Request 600m CPU → Gets 0m CPU initially
   - **Fair share of ChildY**: 400m CPU (already consumed by App2)
   - **Fair share for App3 inside ChildY**: 400m / 2 = 200m CPU per app
   - **Preemption**: App3 preempts 200m CPU from App2 to achieve its fair share
   - **Result**: App2 gets 200m CPU, App3 gets 200m CPU

## Key Validation Points
- Guarantee takes precedence over equal fair share distribution
- Fair share calculation respects queue guarantees: max(fair_share, guarantee)
- Preemption occurs when queues exceed their guarantee-based fair share
- Intra-queue preemption works when multiple apps compete within the same queue
- Final allocation: ChildX gets 600m CPU, ChildY gets 400m CPU total (200m per app)

## Files
- `queues.yaml`: Queue configuration with unequal guarantees (childX=600m, childY=100m)
- `job-app1.yaml`: Job for childX that requests 800m CPU (8 containers × 100m CPU each)
- `job-app2.yaml`: Job for childY that requests 600m CPU (6 containers × 100m CPU each)
- `job-app3.yaml`: Job for childY that requests 600m CPU (6 containers × 100m CPU each)
