# Test Case 7: Preemption with Namespace Wrappers and Dynamic Queue Creation

## Overview
Validates preemption within queues using namespace wrappers and dynamic queue creation with parent queues using childtemplate.

## Test Scenario
```
root.root2: max=1000 CPU
├── groupA: guarantee=500 CPU
│   ├── childX: guarantee=100 CPU (parent with childtemplate)
│   │   └── namespaceA.job-app1
│   └── childY: guarantee=100 CPU (parent with childtemplate)
│       └── namespaceB.{job-app2, job-app3}
└── groupB: guarantee=500 CPU
```

## Expected Behavior

### Phase 1: Initial Allocation
1. **job-app1 → childX**: Request 600 CPU → Gets 600 CPU
2. **job-app2 → childY**: Request 600 CPU → Gets 400 CPU
   - Preemption: childX overusing by 100 CPU → childX gets 500 CPU, childY gets 500 CPU

### Phase 2: Additional Application
3. **job-app3 → childY**: Request 600 CPU
   - Fair share: childY = 500 CPU / 2 = 250 CPU per app
   - Preemption: job-app3 preempts 200 CPU from job-app2
   - Result: job-app2 gets 300 CPU, job-app3 gets 200 CPU

## Key Validation Points
- Fair share calculation with namespace wrappers and dynamic queue creation
- Preemption between applications in dynamically created leaf queues
- Final allocation: job-app2 gets 300 CPU, job-app3 gets 200 CPU in childY

## Files
- `queues.yaml`: Queue configuration with childX and childY as parent queues with childtemplate
- `job-app1.yaml`: Requests 600 CPU
- `job-app2.yaml`: Requests 600 CPU
- `job-app3.yaml`: Requests 600 CPU
