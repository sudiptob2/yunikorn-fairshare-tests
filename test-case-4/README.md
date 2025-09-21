# Test Case 4: Unequal Resource Distribution

## Overview
This test case validates fair share preemption when sibling queues have different resource requirements and guarantees. It tests how the fair share algorithm handles queues with asymmetric resource needs.

## Test Scenario
```
root: 1000 CPU
├── groupA: guarantee=500 CPU
│   ├── childX: guarantee=100 CPU (small workload)
│   └── childY: guarantee=400 CPU (large workload)
└── groupB: guarantee=500 CPU
```

## Expected Behavior

### Phase 1: Initial Allocation
1. **App1 → childX**: Request 200m CPU → Gets 200m CPU (initially available)
2. **App2 → childY**: Request 800m CPU → Gets 800m CPU (initially available)

### Phase 2: Fair Share Calculation and Preemption (Triggered by Demand)
3. **App3 → childX**: Request 300m CPU → Gets 0m CPU initially
   - **Fair share calculation**: total_allocation_of_parent / active_child = 1000 / 2 = 500m CPU per child
   - **ChildX**: Now has 2 apps, so fair share per app = 500 / 2 = 250m CPU per app
   - **App3's fair share**: 250m CPU (it will try to achieve this fair share)
   - **Current state**: ChildX has 200m CPU (App1), ChildY has 800m CPU (App2)
   - **Preemption needed**: ChildY is overusing by 300m CPU (800 - 500 = 300m over fair share)
   - **Preemption**: 300m CPU is preempted from ChildY to give ChildX its fair share
   - **Result**: ChildX gets 500m CPU total, ChildY gets 500m CPU (300m CPU transferred from ChildY to ChildX)
   - **Final allocation**: App1 gets 200m CPU, App3 gets 250m CPU in ChildX; App2 gets 500m CPU in ChildY

## Key Validation Points
- Fair share calculation works correctly: 1000 / 2 = 500m CPU per child
- Per-app fair share calculation works within queues: 500 / 2 = 250m CPU per app in ChildX
- Preemption occurs based on fair share, not just guarantees
- Smaller queues can still get their fair share even against larger siblings

**⚠️ Limitation**: Even though fair share of ChildX is 500m CPU and App1 is using 200m CPU, there is headroom of 300m CPU in ChildX. However, fair share is calculated without considering the headroom, as a result App3 ends up getting 250m CPU even though there was a chance to get 300m CPU. The current fair share algorithm does not handle this case optimally, so there is room for improvement here.

## Files
- `queues.yaml`: Queue configuration with unequal guarantees
- `job-app1.yaml`: Job for childX that requests 200m CPU (2 containers × 100m CPU each)
- `job-app2.yaml`: Job for childY that requests 800m CPU (8 containers × 100m CPU each)
- `job-app3.yaml`: Job for childX that requests 300m CPU (3 containers × 100m CPU each)
