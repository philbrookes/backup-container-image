# Integreatly Backup Openshift Templates

Openshift templates that uses the backup linux container image to run either a job or a cronjob component backup.

## Usage

Both templates requires three secrets objects:

* component secret
* backend secret
* encryption secret

Check the [smaple-config folder](./sample-config) for examples.

### Cronjob

Creates a backup cronjob:

```
oc new-app \
-f backup-cronjob-template.yaml \
-p 'COMPONENT=mysql' \
-p 'COMPONENT_SECRET_NAME=sample-mysql-secret' \
-p 'BACKEND_SECRET_NAME=sample-s3-secret' \
-p 'ENCRYPTION_SECRET_NAME=sample-gpg-secret' \
-p 'IMAGE=quay.io/integreatly/backup-container:master' \
-p 'CRON_SCHEDULE=* */4 * * *' \
-p 'NAME=mysql-backup'
```

Note: There is knonwn issue in openshift 3.11 around using cronjobs in templates where you may get the following error: `error: no kind "CronJob" is registered for version "batch/v1" in scheme "k8s.io/kubernetes/pkg/api/legacyscheme/scheme.go:29"`, you will need to process the template first and then piping process command output to `oc apply` command:

```
oc process \
-f backup-cronjob-template.yaml \
-p 'COMPONENT=mysql' \
-p 'COMPONENT_SECRET_NAME=sample-mysql-secret' \
-p 'BACKEND_SECRET_NAME=sample-s3-secret' \
-p 'ENCRYPTION_SECRET_NAME=sample-gpg-secret' \
-p 'IMAGE=quay.io/integreatly/backup-container:master' \
-p 'CRON_SCHEDULE=* */4 * * *' \
-p 'NAME=mysql-backup' | oc apply -f -
``` 

Parameters:

```
NAME                     DESCRIPTION                                                          GENERATOR           VALUE
NAME                     Unique job name to be used in several resource name(s)                                   integreatly-cronjob-backup
COMPONENT                Component name to run the backup                                                         
BACKEND                  Backend engine to upload the component archive                                           s3
ENCRYPTION               Encryption engine to encrypt component archive before uploading it                       gpg
COMPONENT_SECRET_NAME    Component secret name to create environment variables from                               
BACKEND_SECRET_NAME      Backend secret name to create environment variables from                                 
ENCRYPTION_SECRET_NAME   Encruption secret name to create environment variables from                              
CRON_SCHEDULE            Job schedule in Cron Format [Default is everyday at 2am]                                 */1 * * * *
IMAGE                    Backup docker image URL                                                                  quay.io/integreatly/backup-container:master
DEBUG                    Debug flag to sleep the job pod after its execution  
```

### Job

Creates and runs an onetime job:

```
oc new-app \
-f backup-job-template.yaml \
-p 'COMPONENT=mysql' \
-p 'COMPONENT_SECRET_NAME=sample-mysql-secret' \
-p 'BACKEND_SECRET_NAME=sample-s3-secret' \
-p 'ENCRYPTION_SECRET_NAME=sample-gpg-secret' \
-p 'IMAGE=quay.io/integreatly/backup-container:master' \
-p 'NAME=mysql-backup'
```

Parameters:

```
NAME                     DESCRIPTION                                                          GENERATOR           VALUE
NAME                     Unique job name to be used in several resource name(s)                                   integreatly-job-backup
COMPONENT                Component name to run the backup                                                         
BACKEND                  Backend engine to upload the component archive                                           s3
ENCRYPTION               Encryption engine to encrypt component archive before uploading it                       gpg
COMPONENT_SECRET_NAME    Component secret name to create environment variables from                               
BACKEND_SECRET_NAME      Backend secret name to create environment variables from                                 
ENCRYPTION_SECRET_NAME   Encruption secret name to create environment variables from                              
IMAGE                    Backup docker image URL                                                                  quay.io/integreatly/backup-container:master
DEBUG                    Debug flag to sleep the job pod after its execution
```