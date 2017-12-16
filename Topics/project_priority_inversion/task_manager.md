# Task Manager

The task manager provides a comprehensive set of directives to create, delete, and administer tasks. The directives provided by the task manager are:
- `rtems_task_create` - Create a task
- `rtems_task_ident` - Get ID of a task
- `rtems_task_self` - Obtain ID of caller
- `rtems_task_start` - Start a task
- `rtems_task_restart` - Restart a task
- `rtems_task_delete` - Delete a task
- `rtems_task_suspend` - Suspend a task
- `rtems_task_resume` - Resume a task
- `rtems_task_is_suspended` - Determine if a task is suspended 
- `rtems_task_set_priority` - Set task priority
- `rtems_task_get_priority` - Get task priority
- `rtems_task_mode` - Change current task’s mode
- `rtems_task_wake_after` - Wake up after interval
- `rtems_task_wake_when` - Wake up when specified
- `rtems_task_get_scheduler` - Get scheduler of a task
- `rtems_task_set_scheduler` - Set scheduler of a task
- `rtems_task_get_affinity` - Get task processor affinity
- `rtems_task_set_affinity` - Set task processor affinity
- `rtems_task_iterate` - Iterate Over Tasks

From RTEMS’ perspective, a task is the smallest thread of execution which can compete on its own for system resources.
A task is manifested by the existence of a task control block (TCB).

## Task Name
By default, the task name is defined by the task object name given to `rtems_task_create()`.
The task name can be obtained with the `pthread_getname_np()` function.
Optionally, a new task name may be set with the `pthread_setname_np()` function.
The maximum size of a task name is defined by the application configuration option `CONFIGURE_MAXIMUM_THREAD_NAME_SIZE`.

## Task States
A task may exist in one of the following five states: 
- **executing** - Currently scheduled to the CPU
- **ready** - May be scheduled to the CPU
- **blocked** - Unable to be scheduled to the CPU
- **dormant** - Created task that is not started
- **non-existent** - Uncreated or deleted task

## Task Priority
RTEMS supports 255 levels of priority ranging from 1 to 255.

The data type `rtems_task_priority` is used to store task priorities.

Tasks of numerically smaller priority values are more important tasks than tasks of numerically larger priority values.
For example, a task at priority level 5 is of higher privilege than a task at priority level 10.

There is no limit to the number of tasks assigned to the same priority.

Each task has a priority associated with it at all times.
The initial value of this priority is assigned at task creation time.
The priority of a task may be changed at any subsequent time.

## Task Mode
A task’s execution mode is a combination of the following four components:
- preemption
- ASR processing
- timeslicing
- interrupt level

It is used to modify RTEMS’ scheduling process and to alter the execution environment of the task.
The data type `rtems_task_mode` is used to manage the task execution mode.

### Premption component
The preemption component allows a task to determine when control of the processor is re- linquished.
If preemption is disabled (`RTEMS_NO_PREEMPT`), the task will retain control of the processor as long as it is in the executing state - even if a higher priority task is made ready.
If preemption is enabled (`RTEMS_PREEMPT`) and a higher priority task is made ready, then the processor will be taken away from the current task immediately and given to the higher priority task.

### Timeslicing component
The timeslicing component is used by the RTEMS scheduler to determine how the processor is allocated to tasks of equal priority.
If timeslicing is enabled (`RTEMS_TIMESLICE`), then RTEMS will limit the amount of time the task can execute before the processor is allocated to another ready task of equal priority.
The length of the timeslice is application dependent and specified in the Configuration Table.
If timeslicing is disabled (`RTEMS_NO_TIMESLICE`), then the task will be allowed to execute until a task of higher priority is made ready.
If `RTEMS_NO_PREEMPT` is selected, then the timeslicing component is ignored by the scheduler.

### ASR processing component
The asynchronous signal processing component is used to determine when received signals are to be processed by the task.
If signal processing is enabled (`RTEMS_ASR`), then signals sent to the task will be processed the next time the task executes.
If signal processing is disabled (`RTEMS_NO_ASR`), then all signals received by the task will remain posted until signal processing is enabled.
This component affects only tasks which have established a routine to process asynchronous signals.

### Interrupt Level component
The interrupt level component is used to determine which interrupts will be enabled when the task is executing.
`RTEMS_INTERRUPT_LEVEL(n)` specifies that the task will execute at interrupt level n.


## Task Argument
All RTEMS tasks are invoked with a single argument which is specified when they are started or restarted.

```
 rtems_task user_task(
   rtems_task_argument argument
  );
```

## Directives
### `rtems_task_create` - Create a task
The `rtems_task_create` directive creates a task by allocating a **task control block**, **assigning the task a user-specified name**, **allocating it a stack and floating point context area**, **setting a user- specified initial priority**, **setting a user-specified initial mode**, and **assigning it a task ID**.
Newly created tasks are initially placed in the dormant state.
All RTEMS tasks execute in the most privileged mode of the processor.


### `rtems_task_ident` - Get ID of a task
The task ID may be obtained later using the `rtems_task_ident` directive.
The task ID is used by other directives to manipulate this task.


### `rtems_task_self` - Obtain ID of caller
### `rtems_task_start` - Start a task
The `rtems_task_start` directive is used to place a dormant task in the ready state.
This enables the task to compete, based on its current priority, for the processor and other system resources.

Any actions, such as suspension or change of priority, performed on a task prior to starting it are nullified when the task is started.

With the `rtems_task_start` directive the user specifies the task’s starting address and argument.
The argument is used to communicate some startup information to the task. As part of this directive, RTEMS initializes the task’s stack based upon the task’s initial execution mode and start address.
The starting argument is passed to the task in accordance with the target processor’s calling convention.

### `rtems_task_restart` - Restart a task
The `rtems_task_restart` directive restarts a task at its initial starting address with its original priority and execution mode, but with a possibly different argument.
The new argument may be used to distinguish between the original invocation of the task and subsequent invocations.
The task’s stack and control block are modified to reflect their original creation values.

Although references to resources that have been requested are cleared, resources allocated by the task are NOT automatically returned to RTEMS.

A task cannot be restarted unless it has previously been started (i.e. dormant tasks cannot be restarted).

All restarted tasks are placed in the ready state.

### `rtems_task_delete` - Delete a task
RTEMS provides the `rtems_task_delete` directive to allow a task to delete itself or any other task.
This directive removes **all RTEMS references to the task**, **frees the task’s control block**, **removes it from resource wait queues**, and **deallocates its stack** as well as the **optional floating point context**.

The task’s name and ID become inactive at this time, and any subsequent references to either of them is invalid.
In fact, RTEMS may reuse the task ID for another task which is created later in the application.

**Unexpired delay timers** (i.e. those used by `rtems_task_wake_after` and `rtems_task_wake_when`) and **timeout timers** associated with the task are automatically deleted, however, other resources dynamically allocated by the task are NOT automatically returned to RTEMS.
Therefore, before a task is deleted, all of its dynamically allocated resources should be deallocated by the user.
This may be accomplished by instructing the task to delete itself rather than directly deleting the task.
Other tasks may instruct a task to delete itself by sending a **“delete self”** message, event, or signal, or by restarting the task with special arguments which instruct the task to delete itself.

### `rtems_task_suspend` - Suspend a task
The `rtems_task_suspend` directive is used to place either the caller or another task into a sus- pended state.
The task remains suspended until a `rtems_task_resume` directive is issued.
This implies that a task may be suspended as well as blocked waiting either to acquire a resource or for the expiration of a timer.

### `rtems_task_resume` - Resume a task
The `rtems_task_resume` directive is used to remove another task from the suspended state.
If the task is not also blocked, resuming it will place it in the ready state, allowing it to once again compete for the processor and resources.

If the task was blocked as well as suspended, this directive clears the suspension and leaves the task in the blocked state.

Suspending a task which is already suspended or resuming a task which is not suspended is considered an error.

### `rtems_task_is_suspended` - Determine if a task is suspended 
The `rtems_task_is_suspended` can be used to determine if a task is currently suspended.

### `rtems_task_set_priority` - Set task priority
The `rtems_task_set_priority` directive is used to obtain or change the current priority of either the calling task or another task.
If the new priority requested is `RTEMS_CURRENT_PRIORITY` or the task’s actual priority, then the current priority will be returned and the task’s priority will remain unchanged.
If the task’s priority is altered, then the task will be scheduled according to its new priority.

### `rtems_task_get_priority` - Get task priority

### `rtems_task_mode` - Change current task’s mode
The `rtems_task_mode` directive is used to obtain or change the current execution mode of the calling task

### `rtems_task_wake_after` - Wake up after interval
The `rtems_task_wake_after` directive creates a sleep timer which allows a task to go to sleep for a specified interval.
The task is blocked until the delay interval has elapsed, at which time the task is unblocked.
A task calling the `rtems_task_wake_after` directive with a delay interval of `RTEMS_YIELD_PROCESSOR` ticks will yield the processor to any other ready task of equal or greater priority and remain ready to execute.

### `rtems_task_wake_when` - Wake up when specified
The `rtems_task_wake_when` directive creates a sleep timer which allows a task to go to sleep until a specified date and time.
The calling task is blocked until the specified date and time has occurred, at which time the task is unblocked.

### `rtems_task_get_scheduler` - Get scheduler of a task
### `rtems_task_set_scheduler` - Set scheduler of a task
### `rtems_task_get_affinity` - Get task processor affinity
### `rtems_task_set_affinity` - Set task processor affinity
### `rtems_task_iterate` - Iterate Over Tasks
