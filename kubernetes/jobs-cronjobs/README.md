# Kubernetes Jobs & CronJobs - Complete Theoretical Guide

**Understanding Batch Processing and Scheduled Tasks**

---

## 📚 Table of Contents

1. [What is a Job?](#what-is-job)
2. [Job Patterns](#job-patterns)
3. [Job Completion and Failure](#completion-failure)
4. [Parallel Jobs](#parallel-jobs)
5. [What is a CronJob?](#what-is-cronjob)
6. [CronJob Schedule](#cronjob-schedule)
7. [CronJob Policies](#cronjob-policies)

---

## 🎯 What is a Job?

A **Job** creates one or more pods and ensures that a specified number of them successfully terminate.

### **Simple Analogy:**

```
Job = One-Time Task Assignment

Boss: "Process this batch of files"

Regular Deployment:
  - Runs forever
  - Restarts if stops
  - Never "completes"

Job:
  - Runs once
  - Processes files
  - Exits when done ✓
  - Doesn't restart

Task = Pod
Completion = Exit code 0
```

---

### **Job vs Deployment:**

```
Deployment:
  Purpose: Long-running services
  Behavior: Runs continuously
  Restart: Always restart if pod dies
  Completion: Never completes
  Examples: Web server, API, database

Job:
  Purpose: Run to completion tasks
  Behavior: Runs until success
  Restart: Only if failed (up to limit)
  Completion: Succeeds and stops
  Examples: Batch processing, migrations, backups
```

---

## 🏗️ Creating Jobs

### **Basic Job:**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-processor
spec:
  template:
    spec:
      containers:
      - name: processor
        image: data-processor:1.0
        command: ["python", "process.py"]
      restartPolicy: Never  # Or OnFailure
```

**What happens:**
```
T+0s:   Job created
T+1s:   Job creates pod: data-processor-abc12
T+2s:   Pod starts, runs python process.py
T+60s:  Script processes data
T+61s:  Script exits with code 0 (success)
T+62s:  Pod status: Completed
T+63s:  Job status: Complete (1/1)

Job is done! ✓
Pod remains (status: Completed)
Can view logs: kubectl logs data-processor-abc12
```

---

### **Viewing Jobs:**

```bash
# List jobs
kubectl get jobs

NAME              COMPLETIONS   DURATION   AGE
data-processor    1/1           61s        5m

# COMPLETIONS: successful/desired

# Describe job
kubectl describe job data-processor

Name:           data-processor
Namespace:      default
Completions:    1
Parallelism:    1
Start Time:     Mon, 08 Jan 2024 10:00:00 +0000
Completed At:   Mon, 08 Jan 2024 10:01:01 +0000
Duration:       61s
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed

# Pod created by job
kubectl get pods

NAME                    READY   STATUS      RESTARTS   AGE
data-processor-abc12    0/1     Completed   0          5m

# View logs (pod remains after completion)
kubectl logs data-processor-abc12
```

---

## 📋 Job Patterns

### **1. Single Pod Job (Default):**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: backup
spec:
  completions: 1  # Run 1 pod
  template:
    spec:
      containers:
      - name: backup
        image: backup-tool:1.0
        command: ["./backup.sh"]
      restartPolicy: Never
```

**Behavior:**
```
Create 1 pod
Run to completion
Job succeeds when pod exits 0
```

---

### **2. Multiple Completions (Sequential):**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: batch-processor
spec:
  completions: 5  # Need 5 successful completions
  parallelism: 1  # Run 1 at a time (sequential)
  template:
    spec:
      containers:
      - name: processor
        image: processor:1.0
      restartPolicy: Never
```

**Behavior:**
```
T+0s:   Create pod-1
T+30s:  pod-1 completes successfully (1/5)
T+31s:  Create pod-2
T+61s:  pod-2 completes successfully (2/5)
T+62s:  Create pod-3
T+92s:  pod-3 completes successfully (3/5)
T+93s:  Create pod-4
T+123s: pod-4 completes successfully (4/5)
T+124s: Create pod-5
T+154s: pod-5 completes successfully (5/5)
T+155s: Job complete! ✓

Sequential: One pod at a time
Total: 5 pods run to completion
```

---

### **3. Parallel Processing:**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: parallel-processor
spec:
  completions: 10  # Need 10 successful completions
  parallelism: 3   # Run 3 pods simultaneously
  template:
    spec:
      containers:
      - name: processor
        image: processor:1.0
      restartPolicy: Never
```

**Behavior:**
```
T+0s:   Create 3 pods (pod-1, pod-2, pod-3) simultaneously
T+30s:  pod-1 completes (1/10), create pod-4
T+35s:  pod-2 completes (2/10), create pod-5
T+40s:  pod-3 completes (3/10), create pod-6
T+70s:  pod-4 completes (4/10), create pod-7
...
T+150s: All 10 pods completed
T+151s: Job complete! ✓

Parallel: Up to 3 pods running simultaneously
Faster than sequential!
```

---

### **4. Work Queue Pattern:**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: queue-processor
spec:
  # No completions specified!
  parallelism: 5  # Run 5 workers
  template:
    spec:
      containers:
      - name: worker
        image: queue-worker:1.0
        env:
        - name: QUEUE_URL
          value: "redis://queue:6379"
      restartPolicy: Never
```

**Behavior:**
```
Work queue (Redis): 100 items

T+0s:   Create 5 worker pods
        Each pod:
          1. Connect to queue
          2. Pull item
          3. Process
          4. Mark done
          5. Repeat until queue empty
          6. Exit (code 0)

T+120s: First worker finishes → exits
T+125s: Second worker finishes → exits
...
T+180s: All workers finish (queue empty)
T+181s: Job complete! ✓

Self-coordinating workers
No completions count needed
```

---

## ✅ Job Completion and Failure

### **Success Criteria:**

```yaml
spec:
  completions: 3
  
Job succeeds when:
  3 pods exit with code 0
```

---

### **Failure Handling:**

```yaml
spec:
  backoffLimit: 4  # Max retries (default: 6)
  completions: 1
  template:
    spec:
      containers:
      - name: flaky-app
        image: flaky:1.0
      restartPolicy: Never
```

**Scenario: Pod fails:**
```
T+0s:   Create pod-1
T+10s:  pod-1 fails (exit code 1)
T+11s:  Retry 1/4: Create pod-2
T+21s:  pod-2 fails (exit code 1)
T+22s:  Retry 2/4: Create pod-3
T+32s:  pod-3 succeeds (exit code 0) ✓
T+33s:  Job complete!

If all 4 retries failed:
T+50s:  backoffLimit reached
        Job status: Failed
        No more pods created
```

---

### **restartPolicy Options:**

**Never:**
```yaml
restartPolicy: Never

Pod fails → Job creates NEW pod
pod-1 fails → create pod-2
pod-2 fails → create pod-3

Each failure = new pod
```

**OnFailure:**
```yaml
restartPolicy: OnFailure

Pod fails → Restart SAME pod
pod-1 fails → restart pod-1
pod-1 fails again → restart pod-1 again

Each failure = container restart in same pod
```

---

### **Active Deadline:**

```yaml
spec:
  activeDeadlineSeconds: 600  # Timeout after 10 minutes
  template:
    spec:
      containers:
      - name: processor
        image: processor:1.0
```

**Behavior:**
```
T+0s:   Job starts
T+600s: Deadline reached
        Job terminated (even if not complete)
        Status: Failed
        Reason: DeadlineExceeded

Use case: Time-bound jobs
  Don't want jobs running forever
```

---

## 🔢 Parallel Jobs

### **Controlling Parallelism:**

```yaml
spec:
  completions: 10
  parallelism: 3

# Start: 3 pods
# Each completion: Start another
# Max running: 3
# Total run: 10
```

---

### **Dynamic Scaling:**

```bash
# Scale parallelism during job execution
kubectl scale job batch-processor --replicas=5

# Was: 3 pods running
# Now: 5 pods running (2 more created)
```

---

### **Example: Image Processing:**

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: image-resizer
spec:
  completions: 1000  # 1000 images to process
  parallelism: 20    # 20 workers simultaneously
  template:
    spec:
      containers:
      - name: resizer
        image: image-resizer:1.0
        env:
        - name: IMAGE_QUEUE
          value: "s3://my-bucket/images/"
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
      restartPolicy: OnFailure
```

**Processing:**
```
1000 images in S3 bucket

20 workers created
Each worker:
  - Pulls image from S3
  - Resizes
  - Uploads to output bucket
  - Repeats

Worker completes → New worker created
Until 1000 images processed
```

---

## ⏰ What is a CronJob?

A **CronJob** creates Jobs on a repeating schedule.

### **Simple Analogy:**

```
CronJob = Scheduled Recurring Task

Like cron on Linux:
  - Run backup every night at 2 AM
  - Generate report every Monday at 9 AM
  - Clean logs every hour

CronJob creates Job
Job creates Pod
Pod runs task
```

---

### **CronJob Example:**

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
spec:
  schedule: "0 2 * * *"  # Every day at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: db-backup:1.0
            command: ["./backup.sh"]
            env:
            - name: DB_HOST
              value: "postgres.default.svc"
          restartPolicy: OnFailure
```

**What happens:**
```
Every day at 2:00 AM:
  1. CronJob creates Job
  2. Job creates Pod
  3. Pod runs backup script
  4. Backup completes
  5. Pod status: Completed

Next day at 2:00 AM:
  Process repeats
```

---

## 📅 CronJob Schedule

Cron format: `minute hour day month weekday`

### **Schedule Examples:**

```yaml
# Every hour
schedule: "0 * * * *"

# Every day at 3:30 AM
schedule: "30 3 * * *"

# Every Monday at 9 AM
schedule: "0 9 * * 1"

# Every 15 minutes
schedule: "*/15 * * * *"

# First day of month at midnight
schedule: "0 0 1 * *"

# Every weekday (Mon-Fri) at 8 AM
schedule: "0 8 * * 1-5"

# Every 6 hours
schedule: "0 */6 * * *"
```

**Cron format:**
```
┌───────────── minute (0 - 59)
│ ┌───────────── hour (0 - 23)
│ │ ┌───────────── day of month (1 - 31)
│ │ │ ┌───────────── month (1 - 12)
│ │ │ │ ┌───────────── day of week (0 - 6) (Sunday=0)
│ │ │ │ │
│ │ │ │ │
* * * * *

Special characters:
  *  Any value
  ,  List (1,3,5)
  -  Range (1-5)
  /  Step (*/15 = every 15)
```

---

### **Viewing CronJobs:**

```bash
# List CronJobs
kubectl get cronjobs

NAME              SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
database-backup   0 2 * * *     False     0        8h              30d
report-generator  0 9 * * 1     False     0        2d              30d

# ACTIVE: Currently running jobs
# LAST SCHEDULE: Time since last job run

# Describe CronJob
kubectl describe cronjob database-backup

Name:                       database-backup
Namespace:                  default
Schedule:                   0 2 * * *
Concurrency Policy:         Allow
Suspend:                    False
Successful Job History Limit:  3
Failed Job History Limit:      1
Starting Deadline Seconds:  <unset>
Last Schedule Time:         Mon, 08 Jan 2024 02:00:00 +0000

# View jobs created by CronJob
kubectl get jobs

NAME                        COMPLETIONS   DURATION   AGE
database-backup-28349570    1/1           45s        8h
database-backup-28350850    1/1           43s        32h
database-backup-28352130    1/1           44s        56h
```

---

## ⚙️ CronJob Policies

### **1. Concurrency Policy:**

```yaml
spec:
  concurrencyPolicy: Allow  # Allow / Forbid / Replace
```

**Allow (Default):**
```
Multiple jobs can run concurrently

9:00 AM: Job-1 starts (still running)
9:01 AM: Job-2 starts (scheduled, Job-1 not done)

Both running simultaneously
```

**Forbid:**
```
Skip new job if previous still running

9:00 AM: Job-1 starts
9:01 AM: Scheduled, but Job-1 still running
         Skip this run!
9:02 AM: Job-1 completes
9:03 AM: Job-2 starts (normal schedule)

Prevents overlap
```

**Replace:**
```
Kill old job, start new

9:00 AM: Job-1 starts
9:01 AM: Scheduled, Job-1 still running
         Terminate Job-1
         Start Job-2

Always runs latest scheduled job
```

---

### **2. Starting Deadline:**

```yaml
spec:
  startingDeadlineSeconds: 300  # Must start within 5 minutes
```

**Use case:**
```
Scheduled: 2:00 AM
Controller unavailable: 2:00-2:10 AM
Controller back: 2:10 AM

Without startingDeadlineSeconds:
  Job starts at 2:10 AM (10 min late)

With startingDeadlineSeconds: 300:
  Deadline: 2:05 AM
  Current: 2:10 AM (past deadline)
  Job skipped (counted as missed)

Prevents very late executions
```

---

### **3. History Limits:**

```yaml
spec:
  successfulJobsHistoryLimit: 3  # Keep last 3 successful jobs
  failedJobsHistoryLimit: 1      # Keep last 1 failed job
```

**Cleanup:**
```
CronJob runs daily:
  Day 1: Job-1 (Success) ✓
  Day 2: Job-2 (Success) ✓
  Day 3: Job-3 (Success) ✓
  Day 4: Job-4 (Success) ✓
         Job-1 deleted (keep only 3)
  Day 5: Job-5 (Success) ✓
         Job-2 deleted

Prevents infinite job accumulation
```

---

### **4. Suspend:**

```yaml
spec:
  suspend: true  # Temporarily disable CronJob
```

**Use case:**
```bash
# Suspend CronJob (stop scheduling)
kubectl patch cronjob database-backup -p '{"spec":{"suspend":true}}'

# CronJob suspended
# No new jobs created
# Existing jobs continue

# Resume
kubectl patch cronjob database-backup -p '{"spec":{"suspend":false}}'

# Useful for maintenance windows
```

---

## 📧 Contact Information

**Prepared by:** Manohar Gulme  
**Email:** manohar.gulme@outlook.com  
**Phone:** +91 8919161280  
**LinkedIn:** [linkedin.com/in/manohar-gulme](https://www.linkedin.com/in/manohar-gulme/)

