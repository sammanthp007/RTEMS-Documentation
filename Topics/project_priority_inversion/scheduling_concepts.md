# Task state transition

A task occupies the non-existent state before a `rtems_task_create` has been issued on its behalf.

A task enters the non-existent state from any other state in the system when it is deleted with the `rtems_task_delete` directive.
While a task occupies this state it does not have a TCB or a task ID assigned to it; therefore, no other tasks in the system may reference this task.

When a task is created via the `rtems_task_create` directive it enters the dormant state.
This state is not entered through any other means.
Although the task exists in the system, it cannot actively compete for system resources.

It will remain in the dormant state until it is started via the `rtems_task_start` directive, at which time it enters the ready state.
The task is now permitted to be scheduled for the processor and to compete for other system resources.

A task occupies the blocked state whenever it is unable to be scheduled to run.
A running task may block itself or be blocked by other tasks in the system.
The running task blocks itself through voluntary operations that cause the task to wait.
The only way a task can block a task other than itself is with the `rtems_task_suspend` directive.

A task enters the blocked state due to any of the following conditions:
- A task issues a `rtems_task_suspend` directive which blocks either itself or another task in the system.
- The running task issues a `rtems_barrier_wait` directive.
- The running task issues a `rtems_message_queue_receive` directive with the wait option and the message queue is empty.
- The running task issues an `rtems_event_receive` directive with the wait option and the currently pending events do not satisfy the request.
- The running task issues a `rtems_semaphore_obtain` directive with the wait option and the requested semaphore is unavailable.
- The running task issues a `rtems_task_wake_after` directive which blocks the task for the given time interval. If the time interval specified is zero, the task yields the processor and remains in the ready state.
- The running task issues a `rtems_task_wake_when` directive which blocks the task until the requested date and time arrives.
- The running task issues a `rtems_rate_monotonic_period` directive and must wait for the specified rate monotonic period to conclude.
- The running task issues a `rtems_region_get_segment` directive with the wait option and there is not an available segment large enough to satisfy the task’s request.

A task occupies the ready state when it is able to be scheduled to run, but currently does not have control of the processor.
Tasks of the same or higher priority will yield the processor by either becoming blocked, completing their timeslice, or being deleted.
All tasks with the same priority will execute in FIFO order.
A task enters the ready state due to any of the following conditions:
-  A running task issues a `rtems_task_resume` directive for a task that is suspended and the task is not blocked waiting on any resource.
-  A running task issues a `rtems_message_queue_send`, `rtems_message_queue_broadcast`, or a `rtems_message_queue_urgent` directive which posts a message to the queue on which the blocked task is waiting.
-  A running task issues an `rtems_event_send` directive which sends an event condition to a task which is blocked waiting on that event condition.
-  A running task issues a `rtems_semaphore_release` directive which releases the semaphore on which the blocked task is waiting.
-  A timeout interval expires for a task which was blocked by a call to the `rtems_task_wake_after` directive.
-  A timeout period expires for a task which blocked by a call to the `rtems_task_wake_when` directive.
-  A running task issues a `rtems_region_return_segment` directive which releases a segment to the region on which the blocked task is waiting and a resulting segment is large enough to satisfy the task’s request.
-  A rate monotonic period expires for a task which blocked by a call to the `rtems_rate_monotonic_period` directive.
-  A timeout interval expires for a task which was blocked waiting on a message, event, semaphore, or segment with a timeout specified.
-  A running task issues a directive which deletes a message queue, a semaphore, or a region on which the blocked task is waiting.
-  A running task issues a `rtems_task_restart` directive for the blocked task.
- The running task, with its preemption mode enabled, may be made ready by issuing any of the directives that may unblock a task with a higher priority. This directive may be issued from the running task itself or from an ISR. A ready task occupies the executing state when it has control of the CPU. A task enters the executing state due to any of the following conditions:
- The task is the highest priority ready task in the system.
- The running task blocks and the task is next in the scheduling queue. The task may be of equal priority as in round-robin scheduling or the task may possess the highest priority of the remaining ready tasks.
- The running task may reenable its preemption mode and a task exists in the ready queue that has a higher priority than the running task.
- The running task lowers its own priority and another task is of higher priority as a result.
- The running task raises the priority of a task above its own and the running task is in preemption mode.

## Directives
### SCHEDULER_IDENT - Get ID of a scheduler
Identifies a scheduler by its name.
```
rtems_status_code rtems_scheduler_ident(
    rtems_name  name,
    rtems_id   *id
);
```

### SCHEDULER_IDENT_BY_PROCESSOR - Get ID of a scheduler by processor
Identifies a scheduler by a processor.
```
rtems_status_code rtems_scheduler_ident_by_processor(
  uint32_t cpu_index,
  rtems_id *id
);
```

### SCHEDULER_IDENT_BY_PROCESSOR_SET - Get ID of a scheduler by processor set
Identifies a scheduler by a processor set.
```
rtems_status_code rtems_scheduler_ident_by_processor_set(
  size_t cpusetsize,
  const cpu_set_t *cpuset,
  rtems_id *id
);
```

### SCHEDULER_GET_PROCESSOR_SET - Get processor set of a scheduler
Returns the processor set owned by the scheduler instance in cpuset.
```
rtems_status_code rtems_scheduler_get_processor_set(
  rtems_id scheduler_id,
  size_t cpusetsize,
  cpu_set_t *cpuset
);
```

### SCHEDULER_ADD_PROCESSOR - Add processor to a scheduler
Adds a processor to the set of processors owned by the specified scheduler instance.
```
rtems_status_code rtems_scheduler_add_processor(
  rtems_id scheduler_id,
  uint32_t cpu_index
);
```

### SCHEDULER_REMOVE_PROCESSOR - Remove processor from a scheduler
Removes a processor from set of processors owned by the specified scheduler instance.
```
rtems_status_code rtems_scheduler_remove_processor(
  rtems_id scheduler_id,
  uint32_t cpu_index
);
```
