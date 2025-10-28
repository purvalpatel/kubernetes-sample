Description:
------

- A jobs runs a pod that does a task and then stops.
- It’s mean short-lived, one time tasks - not a long running services.

- its like background worker or a cron job that runs only once.

Use cases:
- Data migration scripts
- Batch processing
- Cleanup tasks
- Sending report or a email
- Machine learning training model 


YAML:
```YAML
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  template:
    spec:
      containers:
      - name: hello
        image: busybox
        command: ["echo", "Hello from the Job"]
      restartPolicy: Never
```

## Cronjob:

- Cronjob create job on repeating schedule.

Use case:
- Backups,
- Report generations

YAML:
```
apiVersion: batch/v1
kind: CronJob
metadata:
  name: hello
spec:
  schedule: "* * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: hello
            image: busybox:1.28
            imagePullPolicy: IfNotPresent
            command:
            - /bin/sh
            - -c
            - date; echo Hello from the Kubernetes cluster
          restartPolicy: OnFailure
```
