* 任务在后台执行或者暂停状态（`ctrl + z`暂停任务）时，执行`fg <job-id>`将可以把对应的任务转到前台继续执行。在前台执行时，无法在console中执行其他命令

* 当任务处于暂停状态时（`ctrl + z`暂停任务），执行`bg <job-id>`将可以把对应的任务在后台继续执行

* 非当前session创建的job，只能由当前session `fg`到前台执行
