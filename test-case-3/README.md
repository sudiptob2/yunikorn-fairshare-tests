# Test Case 3: Full Cluster Preemption

## Overview
This test case validates fair share preemption when the cluster is fully utilized and preemption needs to occur across different groups. It tests complex preemption scenarios involving multiple queue levels and cross-group resource allocation.

## Test Scenario
```
root: 1000 CPU
├── root.GroupA: guarantee=500 CPU
│   ├── root.GroupA.ChildX: guarantee=100 CPU
│   └── root.GroupA.ChildY: guarantee=100 CPU
└── root.GroupB: guarantee=500 CPU
```

## Expected Behavior

### Phase 1: Cluster Full Utilization
1. **App4 → GroupB**: Request 1000 CPU → Gets 1000 CPU (Cluster is full now)

### Phase 2: Cross-Group Preemption
2. **App1 → GroupA.ChildX**: Request 600 CPU → Gets 0 CPU initially
   - **Needs more**: Try preemption → fair share kicks in
   - **Fair share for ChildX**: 1000 CPU / 2 = 500 CPU, try to get 500 from GroupB
   - **Result**: Gets 500 CPU from GroupB, pending 100 CPU

### Phase 3: Intra-Group Preemption
3. **App2 → GroupA.ChildY**: Request 600 CPU → Gets 0 CPU initially
   - **Needs more**: Try preemption → fair share kicks in
   - **Fair share of ChildX**: GroupA / 2 = 500 / 2 = 250 CPU
   - **Result**: Gets 250 CPU from ChildX (200 CPU in test setup due to container requirements)
   - **ChildY allocation**: ChildY gets 250 CPU

### Phase 4: Intra-Queue Preemption
4. **App3 → GroupA.ChildY**: Request 600 CPU → Gets 0 CPU initially
   - **Fair share of ChildY**: 250 CPU, and App2 is using it entirely
   - **Fair share for App3 inside ChildY**: 250 / 2 = 125 CPU
   - **Preemption**: App3 tries to get its fair share from App2
   - **Result**: Gets 125 CPU (100 CPU in test setup due to minimum container requirements)

## Key Validation Points
- Cross-group preemption works when cluster is full
- Fair share calculations work across different queue levels
- Intra-group preemption occurs when siblings compete for resources
- Intra-queue preemption works between applications in the same queue
- Resource allocation respects minimum container requirements (100 CPU per container)
- Final allocation distributes resources fairly across all levels

## Files
- `queues.yaml`: Queue configuration with GroupA and GroupB
- `job-app1.yaml`: Job for GroupA.ChildX that requests 600 CPU
- `job-app2.yaml`: Job for GroupA.ChildY that requests 600 CPU
- `job-app3.yaml`: Additional job for GroupA.ChildY that requests 600 CPU
- `job-app4.yaml`: Job for GroupB that requests 1000 CPU (fills cluster)
