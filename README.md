# Yunikorn Fairshare Tests

Collection of test cases for validating Yunikorn fairshare preemption behavior.

## Limitation

Yunikorn has a limitation where it cannot preempt applications within the same queue. This creates challenges when multiple applications compete for resources in a single queue, as fair share calculations cannot be enforced through preemption at the application level.

**Workaround: Dynamic Queues**
This limitation is worked around by using dynamic queues, where each application is wrapped in its own dynamically created queue. This approach allows Yunikorn's fair share preemption to work effectively by treating each application as if it's in its own separate queue, enabling proper resource allocation and preemption based on fair share calculations.

All test cases in this repository use this dynamic queue pattern with `parent: true` configuration and `childtemplate` properties to enable per-application queue creation.

## Test Cases

- **[Test Case 1: Basic Fairshare Preemption](test-case-1/)** - Basic fairshare preemption between sibling queues
- **[Test Case 2: Preemption Inside Same Queue](test-case-2/)** - Preemption within the same queue with multiple applications
- **[Test Case 3: Full Cluster Preemption](test-case-3/)** - Complex preemption when cluster is full, involving cross-group preemption
- **[Test Case 4: Unequal Resource Distribution](test-case-4/)** - Fairshare preemption with asymmetric resource guarantees
- **[Test Case 5: Capped by gurantee](test-case-5/)** - Interaction between guarantee-based and fairshare-based preemption
