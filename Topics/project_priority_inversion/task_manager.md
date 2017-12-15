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
- `rtems_task_is_suspended` - Determine if a task is suspended • rtems_task_set_priority - Set task priority
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
