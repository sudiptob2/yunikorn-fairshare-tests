# Test Case 9: Preemption with Unequal Guarantees and Over-Requesting (Prod-like Config)

## Overview
This test case validates preemption behavior when an application requests more resources than its queue's guarantee, and how fair share preemption works when multiple applications compete within the same queue hierarchy. It tests scenarios where applications over-request resources and how the scheduler handles allocation and preemption.

## Test Scenario
```
root:
└── root.root2: max=1000 CPU
    ├── root.root2.groupA: guarantee=700 CPU
    │   ├── root.root2.groupA.childX: guarantee=200 CPU (parent queue with childtemplate)
    │   │   └── root.root2.groupA.childX.namespaceA (dynamically created)
    │   │       └── root.root2.groupA.childX.namespaceA.job-app1 (leaf queue)
    │   └── root.root2.groupA.childY: guarantee=500 CPU (parent queue with childtemplate)
    │       └── root.root2.groupA.childY.namespaceB (dynamically created)
    │           ├── root.root2.groupA.childY.namespaceB.job-app2 (leaf queue)
    │           └── root.root2.groupA.childY.namespaceB.job-app3 (leaf queue)
    └── root.root2.groupB: guarantee=300 CPU (parent queue with childtemplate)
```

## Expected Behavior

### Phase 1: Initial Allocation - Over-Requesting Resources
1. **job-app1 → groupA.childX.namespaceA.job-app1**: 
   - **Request**: 1000 CPU (10 containers × 100m CPU each)
   - **Queue guarantee**: childX has 200 CPU guarantee (under groupA with 700 CPU guarantee)
   - **Behavior**: job-app1 requests 5× more than childX's guarantee
   - **Expected**: job-app1 should get 1000 because no other resources exist in the cluster

### Phase 2: Additional Applications in Same Queue Hierarchy
2. **job-app2 → groupA.childY.namespaceB.job-app2**: 
   - **Request**: 200 CPU (2 containers × 100m CPU each)
   - **Queue**: childY has 500 CPU guarantee (under groupA with 700 CPU guarantee)
   - **Expected**: job-app2 gets 200 
   
3. **job-app3 → groupA.childY.namespaceB.job-app3**: 
   - **Request**: 1000 CPU (10 containers × 100m CPU each)
   - **Queue**: Same child queue (childY.namespaceB) as job-app2
   - **Queue guarantee**: childY has 500 CPU guarantee
   - **Behavior**: job-app3 requests 1000 CPU but should be capped by childY's guarantee
   - **Expected**: job-app3 gets 300 CPU (capped at childY's guarantee of 500 CPU minus job-app2's 200 CPU = 300 CPU available)
   - **Total allocation in childY**: job-app2 gets 200 CPU + job-app3 gets 300 CPU = 500 CPU (matches childY's guarantee)

**Overall 500 is preemted from ChildX**

## Key Validation Points
- Applications can request more resources than their queue's guarantee (job-app1 requests 1000 CPU vs childX's 200 CPU guarantee)
- Applications requesting more than available guarantee get capped at the queue's guarantee (job-app3 requests 1000 CPU but gets capped at 300 CPU due to childY's 500 CPU guarantee minus job-app2's 200 CPU)
- Preemption behavior when total requests exceed queue guarantees
- Fair share calculation works correctly when multiple applications compete within the same queue hierarchy
- Resource allocation respects queue guarantees:
  - groupA: 700 CPU guarantee (childX: 200 CPU + childY: 500 CPU)
  - groupB: 300 CPU guarantee
- Preemption can occur between applications in dynamically created leaf queues under the same parent

## Files
- `queues.yaml`: Queue configuration with root2 having groupA (guarantee=700 CPU) containing childX (guarantee=200 CPU) and childY (guarantee=500 CPU), and groupB (guarantee=300 CPU) as parent queues with childtemplate
- `job-app1.yaml`: Job for groupA.childX.namespaceA.job-app1 that requests 1000 CPU (10 containers)
- `job-app2.yaml`: Job for groupA.childY.namespaceB.job-app2 that requests 200 CPU (2 containers)
- `job-app3.yaml`: Job for groupA.childY.namespaceB.job-app3 that requests 1000 CPU (10 containers) but gets capped at 300 CPU by childY's guarantee
