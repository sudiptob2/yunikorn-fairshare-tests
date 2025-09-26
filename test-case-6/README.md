# Test Case 6: Continuous Rebalancing

## Overview
This test case validates continuous rebalancing behavior when jobs come and go dynamically.

## Test Scenario
```
root: 1000 CPU (false root - maximum limit)
├── groupA: no guarantee specified
│   ├── childX: no guarantee specified
│   └── childY: no guarantee specified
└── groupB: no guarantee specified
```

## Expected Behavior

### Phase 1: Initial Allocation
1. **App1 → childX**: Request 1000m CPU → Gets 1000m CPU (initially available, long-running job)

### Phase 2: Fair Share Preemption
2. **App2 → childX**: Request 500m CPU → Gets 0m CPU initially
   - **Fair share calculation**: 1000m CPU ÷ 2 active apps = 500m CPU per app
   - **Preemption needed**: App1 is using 1000m CPU (over fair share by 500m CPU)
   - **Preemption**: 500m CPU is preempted from App1 to give App2 its fair share
   - **Result**: App1 gets 500m CPU, App2 gets 500m CPU
   - **Duration**: App2 runs for 60 seconds (short job)

### Phase 3: Resource Recovery
3. **App2 completion**: After 60 seconds, App2 completes and releases its 500m CPU
   - **Resource recovery**: App1's preempted resources get rescheduled
   - **Final state**: App1 gets back to 1000m CPU (full allocation restored)


## Files
- `queues.yaml`: Queue configuration with false root and no guarantees
- `job-app1.yaml`: Long-running job for childX that requests 1000m CPU
- `job-app2.yaml`: Short-running job for childX that requests 500m CPU (runs for 60s)

