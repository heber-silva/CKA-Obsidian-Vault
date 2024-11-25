A CronJob starts one-time Jobs on a repeating schedule.

FEATURE STATE: `Kubernetes v1.21 [stable]`

A _CronJob_ creates [Jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) on a repeating schedule.

CronJob is meant for performing regular scheduled actions such as backups, report generation, and so on. One CronJob object is like one line of a _crontab_ (cron table) file on a Unix system. It runs a Job periodically on a given schedule, written in [Cron](https://en.wikipedia.org/wiki/Cron) format.

CronJobs have limitations and idiosyncrasies. For example, in certain circumstances, a single CronJob can create multiple concurrent Jobs. See the [](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/#cron-job-limitations) below.

When the control plane creates new Jobs and (indirectly) Pods for a CronJob, the `.metadata.name` of the CronJob is part of the basis for naming those Pods. The name of a CronJob must be a valid [](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-subdomain-names) value, but this can produce unexpected results for the Pod hostnames. For best compatibility, the name should follow the more restrictive rules for a [](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#dns-label-names). Even when the name is a DNS subdomain, the name must be no longer than 52 characters. This is because the CronJob controller will automatically append 11 characters to the name you provide and there is a constraint that the length of a Job name is no more than 63 characters.