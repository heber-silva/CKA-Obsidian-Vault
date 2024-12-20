[Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) represent one-off tasks that run to completion and then stop.

A Job creates one or more Pods and will continue to retry execution of the [Pod](../Pod.md)s until a specified number of them successfully terminate. As pods successfully complete, the Job tracks the successful completions. When a specified number of successful completions is reached, the task (ie, Job) is complete. Deleting a Job will clean up the Pods it created. Suspending a Job will delete its active Pods until the Job is resumed again.

A simple case is to create one Job object in order to reliably run one Pod to completion. The Job object will start a new Pod if the first Pod fails or is deleted (for example due to a node hardware failure or a node reboot).

You can also use a Job to run multiple Pods in parallel.

If you want to run a Job (either a single task, or several in parallel) on a schedule, see[CronJob](CronJob.md) (https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/).