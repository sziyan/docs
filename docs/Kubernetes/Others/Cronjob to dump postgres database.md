---
share: true
tags:
created: 2025-09-25
modified: 2025-09-25
---

# Cronjob to dump postgres database

## Retrieve database information

1. Retrieve the username and password of the postgres cluster (e.g from Kubernetes secrets)
2. Either spin up a postgres image, or exec to the existing postgres instance and run the following command:
```
export PGPASSWORD=password
psql -U postgres -h localhost
```
   The above command will attempt to connect to the postgres database with the user “postgres” on localhost which in default installation is a superadmin user.
   3. Run below command to list the tables to backup in the database
	  `\l`

## Create CronJob to backup database
Requirement:
- Ensure a PVC is already created which will store the database dumps


1. Create a Cronjob with below yaml:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
name: pgdump-cron
namespace: runai-backend
spec:
  schedule: '@daily'
  jobTemplate:
    spec:
      template:
	    spec:
		  restartPolicy: Never
		  serviceAccountName: runai-backend-postgresql
		  imagePullSecrets:
			- name: runai-reg-creds
		  enableServiceLinks: true
		  securityContext:
		    fsGroup: 1001
			fsGroupChangePolicy: Always
			seccompProfile:
			  type: RuntimeDefault
		  containers:
			- resources:
				limits:
				  cpu: 512m
				  memory: 512Mi
			name: postgresql
			command: ["/bin/sh", "-c"]
			args: ["pg_dump -U postgres -h runai-backend-postgresql.runai-backend.svc.cluster.local backend > /postgres-backup/backend_dump.dump && echo 'backup successful!'"]
			env:
			- name: PGPASSWORD
			  valueFrom:
			    secretKeyRef:
			      name: runai-backend-postgresql
			      key: password
		    securityContext:
		      runAsGroup: 0
		      runAsUser: 1001
		      seccompProfile:
		        type: RuntimeDefault
		      readOnlyRootFilesystem: false
		      runAsNonRoot: true
		      privileged: false
		      capabilities:
		        drop:
		        - ALL
		      allowPrivilegeEscalation: false
		    imagePullPolicy: IfNotPresent
		    volumeMounts:
		    - name: runai-postgres-backup
		      mountPath: /postgres-backup
		    image: 'mirror.ocpd.mlopslab.ncs:8443/postgresql:2.21.38'
		    automountServiceAccountToken: false
		    serviceAccount: runai-backend-postgresql
		    volumes:
		    - name: runai-postgres-backup
		      persistentVolumeClaim:
			    claimName: runai-postgres-backup
```
A Cronjob is created that will create a pod with postges containter image, run a pg_dump by connecting to the database endpoint, and create the dump of backend table in `/postgres-backup` folder.