

# NeoLoad Web OpenShift template

_This repository has been deprecated. Please contact your sales representative if you want to deploy NeoLoad Web on OpenShift._

## NeoLoad Web Overview

[NeoLoad Web](https://www.neotys.com/neoload/features/neoload-web) is the centralized Performance Testing Platform designed for Continuous Testing :
* Launch performance tests
* Select your load generation infrastructure
* Analyze test results in real time or for terminated tests
* Customize dashboards, based on custom graphs
* Share performance test results with Dev, QA, Ops
* Connect performance testing data: import and correlate third party data / export NeoLoad test data

## Contents
- [Reference Links](#reference-links)
- [Prerequisites](#prerequisites)
- [User guide](#user-guide)
	- [Parameters](#parameters)
	- [Deployment](#deployment)
	- [Checking startup](#checking-startup)
	- [Accessing NeoLoad Web](#accessing-neoload-web)
	- [Getting NeoLoad Web logs](#getting-neoload-web-logs)
	- [Undeploy](#undeploy)
- [Troubleshooting](#troubleshooting)




## Reference links

* [NeoLoad Web documentation](https://www.neotys.com/documents/doc/nlweb/latest/en/html/#2983.htm)
* [NeoLoad Web Frontend Docker image](https://hub.docker.com/repository/docker/neotys/neoload-web-frontend)
* [NeoLoad Web Backend Docker image](https://hub.docker.com/repository/docker/neotys/neoload-web-backend)
* [NeoLoad Web SaaS](https://neoload.saas.neotys.com/)



## Prerequisites

Prerequisites for deployment:

* A running OpenShift cluster
* `oc` command line tool installed and configured to control OpenShift cluster
* A running Mongo DB server or cluster

## User guide

### Parameters

The deployment is based on an OpenShift template that defines the following parameters.

| Parameter  | Description  | Default value  | Required |
|---|---|---|---|
| NLW_VERSION | NeoLoad Web version, it must be a valid Docker image tag from [Neotys Docker hub registry.](https://hub.docker.com/repository/docker/neotys/neoload-web-backend/tags?page=1)  | latest  | false  |
| MONGODB_URL | URL of you Mongo DB server or cluster. Expected format is: `<host>:<port>/<auth database>?<parameters>` example `mongo.mycompany.com:27017/admin?ssl=true`  |  - | true  |
| MONGODB_LOGIN | User login used to authenticate on Mongo DB. Leave it empty if no authentication is required.  | -  | false  |
| MONGODB_PASSWORD | User password used to authenticate on Mongo DB. Leave it empty if no authentication is required.  | -  | false  |
| SECRET_KEY | Passphrase used to encrypt and store sensitive data used in NeoLoad Web (like git passwords). Must be 8 characters minimum.  | -  | true  |
| OPENSHIFT_ROUTE_HOSTNAME | Base hostname used by OpenShift route, like myopenshift.mycompany.com.  |  - | true |


### Deployment

You can deploy NeoLoad Web by running the following command:

```shell
oc process -f ./neoload-web-template.yaml -p NLW_VERSION=latest -p MONGODB_URL=mongo.mycompany.com:27017/admin -p SECRET_KEY=azertyui -p OPENSHIFT_ROUTE_HOSTNAME=openshift.mycompany.com | oc apply -f -
```
#### Output example
```shell
user@host:~$ oc process -f ./neoload-web-template.yaml -p NLW_VERSION=latest -p MONGODB_URL=mongo.mycompany.com:27017/admin -p SECRET_KEY=azertyui -p OPENSHIFT_ROUTE_HOSTNAME=openshift.mycompany.com | oc apply -f -
deploymentconfig.apps.openshift.io/neoload-web created
service/neoload-api-svc created
route.route.openshift.io/neoload-api created
service/neoload-files-svc created
route.route.openshift.io/neoload-files created
service/neoload-front-svc created
route.route.openshift.io/neoload-front created
service/neoload-front-admin-svc created
route.route.openshift.io/neoload-front-admin created
service/neoload-back-admin-svc created
route.route.openshift.io/neoload-back-admin created

```
### Checking startup

You can check the deployment progress by using the following command line `oc get pods --watch`.

#### Output example
```shell
user@host:~$ oc get pods --watch
NAME                   READY   STATUS             RESTARTS    AGE
neoload-web-1-deploy   0/1     Pending            0           0s
neoload-web-1-deploy   0/1     Pending            0           0s
neoload-web-1-deploy   0/1     ContainerCreating  0           0s
neoload-web-1-deploy   1/1     Running            0           4s
neoload-web-1-gxb4r    0/2     Pending            0           0s
neoload-web-1-gxb4r    0/2     Pending            0           0s
neoload-web-1-gxb4r    0/2     ContainerCreating  0           0s
neoload-web-1-gxb4r    0/2     Running            0           6s
neoload-web-1-gxb4r    1/2     Running            0           96s
neoload-web-1-gxb4r    2/2     Running            0           98s
```
This command line will watch Pods startup and status changes. NeoLoad Web is started when the last line (`neoload-web-1-gxb4r   2/2   Running   0     98s`) appears.

If NeoLoad Web doesn't start after a few minutes [take a look at the Troubleshooting section](#troubleshooting).

### Accessing NeoLoad Web
The provided OpenShift template will deploy [OpenShift Routes](https://docs.openshift.com/container-platform/3.11/architecture/networking/routes.html). These Routes will expose NeoLoad Web application.
#### Getting the created Routes
You can get the created Routes by running the following command `oc get routes`.
##### Output example
```shell
user@host:~$ oc get routes
NAME                  HOST/PORT                                                     PATH      SERVICES                  PORT      TERMINATION   WILDCARD
neoload-api           neoload-api-my-workspace.openshift.mycompany.com                        neoload-api-svc           <all>                   None
neoload-back-admin    neoload-back-admin-my-workspace.openshift.mycompany.com                 neoload-back-admin-svc    <all>                   None
neoload-files         neoload-files-my-workspace.openshift.mycompany.com                      neoload-files-svc         <all>                   None
neoload-front         neoload-front-my-workspace.openshift.mycompany.com                      neoload-front-svc         <all>                   None
neoload-front-admin   neoload-front-admin-my-workspace.openshift.mycompany.com                neoload-front-admin-svc   <all>                   None

```
* neoload-api -> neoload-api-my-workspace.openshift.mycompany.com
	* URL exposing NeoLoad Web API.
* neoload-front -> neoload-front-my-workspace.openshift.mycompany.com
	* URL exposing NeoLoad Web UI.
* neoload-files -> neoload-files-my-workspace.openshift.mycompany.com
	* URL exposing NeoLoad Web Files API.
* neoload-front-admin -> neoload-front-admin-my-workspace.openshift.mycompany.com
* neoload-back-admin -> neoload-back-admin-my-workspace.openshift.mycompany.com

### Getting NeoLoad Web logs
In order to get NeoLoad Web logs you should first retrieve the created pod name, you can obtain it by running the following comand `oc get pods`.
```shell
user@host:~$ oc get pods
NAME                  READY     STATUS    RESTARTS   AGE
neoload-web-1-chkvs   2/2       Running   0          19h
```
This pod contains 2 containers respectively named *neoload-web-frontend* and *neoload-web-backend*.
You can get logs from each container by running the following command `oc logs <pod-name> -c <container-name>`.
#### Output examples
##### NeoLoad Web Frontend logs
```
user@host:~$ oc logs neoload-web-1-chkvs -c neoload-web-frontend
Starting mode START_MODE=ON_PREMISE
Starting Jetty on port 9090

Starting tiny server in admin mode on port 9091
Memory Allocation: Xmx1000m
Vertx IP Bus Address: 10.131.1.25 public host 10.131.1.25
13:22:32,493 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Found resource [conf/logback-prod.xml] at [file:/server/conf/logback-prod.xml]
13:22:32,558 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - debug attribute not set
13:22:32,566 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - Will scan for changes in [file:/server/conf/logback-prod.xml]
13:22:32,566 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - Setting ReconfigureOnChangeTask scanning period to 30 seconds
13:22:32,572 |-ERROR in ch.qos.logback.core.joran.action.ShutdownHookAction - Missing class name for shutdown hook. Near [shutdownHook] line 2
13:22:32,572 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - About to instantiate appender of type [ch.qos.logback.core.ConsoleAppender]
13:22:32,575 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - Naming appender as [STDOUT_PROD]
13:22:32,583 |-INFO in ch.qos.logback.core.joran.action.NestedComplexPropertyIA - Assuming default type [ch.qos.logback.classic.encoder.PatternLayoutEncoder] for [encoder] property
13:22:32,637 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - About to instantiate appender of type [ch.qos.logback.classic.AsyncAppender]
13:22:32,638 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - Naming appender as [ASYNC_PROD]
13:22:32,638 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [STDOUT_PROD] to ch.qos.logback.classic.AsyncAppender[ASYNC_PROD]
13:22:32,638 |-INFO in ch.qos.logback.classic.AsyncAppender[ASYNC_PROD] - Attaching appender named [STDOUT_PROD] to AsyncAppender.
13:22:32,639 |-INFO in ch.qos.logback.classic.AsyncAppender[ASYNC_PROD] - Setting discardingThreshold to 51
13:22:32,639 |-INFO in ch.qos.logback.classic.joran.action.LoggerAction - Setting level of logger [com.neotys] to WARN
13:22:32,639 |-INFO in ch.qos.logback.classic.joran.action.LoggerAction - Setting additivity of logger [com.neotys] to false
13:22:32,639 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [ASYNC_PROD] to Logger[com.neotys]
13:22:32,639 |-INFO in ch.qos.logback.classic.joran.action.RootLoggerAction - Setting level of ROOT logger to INFO
13:22:32,640 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [ASYNC_PROD] to Logger[ROOT]
13:22:32,640 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - End of configuration.
13:22:32,640 |-INFO in ch.qos.logback.classic.joran.JoranConfigurator@1794d431 - Registering current configuration as safe fallback point
....
```
##### NeoLoad Web Backend logs
```
user@host:~$ oc logs neoload-web-1-chkvs -c neoload-web-backend
Starting mode START_MODE=ON_PREMISE
Starting Jetty on port 9092

Memory Allocation: Xmx2000m
Vertx IP Bus Address: 10.131.1.25 public host 10.131.1.25
13:22:28,707 |-INFO in ch.qos.logback.classic.LoggerContext[default] - Found resource [conf/logback-prod.xml] at [file:/server/conf/logback-prod.xml]
13:22:28,771 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - debug attribute not set
13:22:28,779 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - Will scan for changes in [file:/server/conf/logback-prod.xml]
13:22:28,779 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - Setting ReconfigureOnChangeTask scanning period to 30 seconds
13:22:28,785 |-ERROR in ch.qos.logback.core.joran.action.ShutdownHookAction - Missing class name for shutdown hook. Near [shutdownHook] line 2
13:22:28,785 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - About to instantiate appender of type [ch.qos.logback.core.ConsoleAppender]
13:22:28,787 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - Naming appender as [STDOUT_PROD]
13:22:28,793 |-INFO in ch.qos.logback.core.joran.action.NestedComplexPropertyIA - Assuming default type [ch.qos.logback.classic.encoder.PatternLayoutEncoder] for [encoder] property
13:22:28,839 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - About to instantiate appender of type [ch.qos.logback.classic.AsyncAppender]
13:22:28,840 |-INFO in ch.qos.logback.core.joran.action.AppenderAction - Naming appender as [ASYNC_PROD]
13:22:28,840 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [STDOUT_PROD] to ch.qos.logback.classic.AsyncAppender[ASYNC_PROD]
13:22:28,840 |-INFO in ch.qos.logback.classic.AsyncAppender[ASYNC_PROD] - Attaching appender named [STDOUT_PROD] to AsyncAppender.
13:22:28,840 |-INFO in ch.qos.logback.classic.AsyncAppender[ASYNC_PROD] - Setting discardingThreshold to 51
13:22:28,841 |-INFO in ch.qos.logback.classic.joran.action.LoggerAction - Setting level of logger [com.neotys] to INFO
13:22:28,841 |-INFO in ch.qos.logback.classic.joran.action.LoggerAction - Setting additivity of logger [com.neotys] to false
13:22:28,841 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [ASYNC_PROD] to Logger[com.neotys]
13:22:28,841 |-INFO in ch.qos.logback.classic.joran.action.RootLoggerAction - Setting level of ROOT logger to INFO
13:22:28,841 |-INFO in ch.qos.logback.core.joran.action.AppenderRefAction - Attaching appender named [ASYNC_PROD] to Logger[ROOT]
13:22:28,841 |-INFO in ch.qos.logback.classic.joran.action.ConfigurationAction - End of configuration.
13:22:28,842 |-INFO in ch.qos.logback.classic.joran.JoranConfigurator@1794d431 - Registering current configuration as safe fallback point
...
```

### Undeploy
You can undeploy NeoLoad Web by running the following command.
```shell
oc process -f ./neoload-web-template.yaml -p NLW_VERSION=latest -p MONGODB_URL=mongo.mycompany.com:27017/admin -p SECRET_KEY=azertyui -p OPENSHIFT_ROUTE_HOSTNAME=openshift.mycompany.com | oc delete -f -
```
#### Output example
```shell
user@host:~$ oc process -f ./neoload-web-template.yaml -p NLW_VERSION=latest -p MONGODB_URL=mongo.mycompany.com:27017/admin -p SECRET_KEY=azertyui -p OPENSHIFT_ROUTE_HOSTNAME=openshift.mycompany.com | oc delete -f -
deploymentconfig.apps.openshift.io "neoload-web" deleted
service "neoload-api-svc" deleted
route.route.openshift.io "neoload-api" deleted
service "neoload-files-svc" deleted
route.route.openshift.io "neoload-files" deleted
service "neoload-front-svc" deleted
route.route.openshift.io "neoload-front" deleted
service "neoload-front-admin-svc" deleted
route.route.openshift.io "neoload-front-admin" deleted
service "neoload-back-admin-svc" deleted
route.route.openshift.io "neoload-back-admin" deleted

```

## Troubleshooting
You will find in this section some clues to help you understand why NeoLoad Web doesn't start.
### NeoLoad Web is not running after deployment
Deployment has been succesfully done but only `neoload-web-1-deploy` Pod is running.
`neoload-web-1-deploy` Pod, is a [deployer Pod](https://docs.openshift.com/container-platform/3.11/architecture/core_concepts/deployments.html#deployments-and-deployment-configurations) started by OpenShift that will manage the NeoLoad Web deployment.

#### Ouput example
```shell
user@host:~$ oc process -f ./neoload-web-template.yaml -p NLW_VERSION=latest -p MONGODB_URL=mongo.mycompany.com:27017/admin -p SECRET_KEY=azertyui -p OPENSHIFT_ROUTE_HOSTNAME=openshift.mycompany.com | oc apply -f -deploymentconfig.apps.openshift.io/neoload-web created
service/neoload-api-svc created
route.route.openshift.io/neoload-api created
service/neoload-files-svc created
route.route.openshift.io/neoload-files created
service/neoload-front-svc created
route.route.openshift.io/neoload-front created
service/neoload-front-admin-svc created
route.route.openshift.io/neoload-front-admin created
service/neoload-back-admin-svc created
route.route.openshift.io/neoload-back-admin created

user@host:~$ oc get pods
NAME                   READY     STATUS    RESTARTS   AGE
neoload-web-1-deploy   1/1       Running   0          1m

user@host:~$ oc get dc
NAME          REVISION   DESIRED   CURRENT   TRIGGERED BY
neoload-web   1          1         0         config
```
#### Debugging
It means that OpenShift was not able to create the neoload-web Pod correctly. To get more information about what is going on we will look at OpenShift events using `oc get events` command.

```shell
user@host:~$ oc get events
LAST SEEN   FIRST SEEN   COUNT     NAME                                    KIND                    SUBOBJECT                     TYPE      REASON              SOURCE                        MESSAGE
23s         23s          1         neoload-web-1-deploy.1603ce30016a093e   Pod                                                   Normal    Scheduled           default-scheduler             Successfully assigned my-workspace/neoload-web-1-deploy to rdopenshift01
23s         23s          1         neoload-web.1603ce2ffa39dacc            DeploymentConfig                                      Normal    DeploymentCreated   deploymentconfig-controller   Created new replication controller "neoload-web-1" for version 1
20s         20s          1         neoload-web-1-deploy.1603ce30c2860d58   Pod                     spec.containers{deployment}   Normal    Created             kubelet, rdopenshift01        Created container
20s         20s          1         neoload-web-1-deploy.1603ce30d800af4f   Pod                     spec.containers{deployment}   Normal    Started             kubelet, rdopenshift01        Started container
20s         20s          1         neoload-web-1-deploy.1603ce30b53fc21f   Pod                     spec.containers{deployment}   Normal    Pulled              kubelet, rdopenshift01        Container image "docker.io/openshift/origin-deployer:v3.11.0" already present on machine
17s         17s          1         neoload-web-1.1603ce31837d47e8          ReplicationController                                 Warning   FailedCreate        replication-controller        Error creating: pods "neoload-web-1-n4klf" is forbidden: maximum cpu usage per Pod is 1, but limit is 4.
17s         17s          1         neoload-web-1.1603ce318536a0f6          ReplicationController                                 Warning   FailedCreate        replication-controller        Error creating: pods "neoload-web-1-m9ktt" is forbidden: maximum cpu usage per Pod is 1, but limit is 4.
17s         17s          1         neoload-web-1.1603ce3185f39d9c          ReplicationController                                 Warning   FailedCreate        replication-controller        Error creating: pods "neoload-web-1-ghhr7" is forbidden: maximum cpu usage per Pod is 1, but limit is 4.
17s         17s          1         neoload-web-1.1603ce31895b42ef          ReplicationController                                 Warning   FailedCreate        replication-controller        Error creating: pods "neoload-web-1-kjrpn" is forbidden: maximum cpu usage per Pod is 1, but limit is 4.
17s         17s          1         neoload-web-1.1603ce31864d86a1          ReplicationController                                 Warning   FailedCreate        replication-controller        Error creating: pods "neoload-web-1-lslfl" is forbidden: maximum cpu usage per Pod is 1, but limit is 4.
16s         16s          1         neoload-web-1.1603ce318e5ad9bc          ReplicationController                                 Warning   FailedCreate        replication-controller        Error creating: pods "neoload-web-1-p6t7p" is forbidden: maximum cpu usage per Pod is 1, but limit is 4.
16s         16s          1         neoload-web-1.1603ce31982722fb          ReplicationController                                 Warning   FailedCreate        replication-controller        Error creating: pods "neoload-web-1-h8rcq" is forbidden: maximum cpu usage per Pod is 1, but limit is 4.
16s         16s          1         neoload-web-1.1603ce31ab7d4035          ReplicationController                                 Warning   FailedCreate        replication-controller        Error creating: pods "neoload-web-1-4sz6j" is forbidden: maximum cpu usage per Pod is 1, but limit is 4.
15s         15s          1         neoload-web-1.1603ce31d1e5d97b          ReplicationController                                 Warning   FailedCreate        replication-controller        Error creating: pods "neoload-web-1-v4tld" is forbidden: maximum cpu usage per Pod is 1, but limit is 4.
6s          14s          3         neoload-web-1.1603ce321e71b181          ReplicationController                                 Warning   FailedCreate        replication-controller        (combined from similar events): Error creating: pods "neoload-web-1-xl29d" is forbidden: maximum cpu usage per Pod is 1, but limit is 4.

```
##### CPU limits
Events are saying `Error creating: pods "neoload-web-1-ghhr7" is forbidden: maximum cpu usage per Pod is 1, but limit is 4.`. It means that [Limit Ranges](https://docs.openshift.com/container-platform/3.11/dev_guide/compute_resources.html#dev-limit-ranges) are defined on your current project.

You can check that using the command `oc get limits`.

```shell
user@host:~$ oc get limits
NAME                   CREATED AT
core-resource-limits   2020-03-27T09:28:44Z

user@host:~$ oc describe limits core-resource-limits
Name:       core-resource-limits
Namespace:  my-workspace
Type        Resource  Min  Max  Default Request  Default Limit  Max Limit/Request Ratio
----        --------  ---  ---  ---------------  -------------  -----------------------
Pod         cpu       -    1    -                -              -
Container   cpu       -    -    100m             100m           -
```

Here we can see that a Pod cannot declare a CPU limit higher than 1.
NeoLoad Web deployment declares a Pod containing two containers having each a CPU limit of 2. So the entire Pod has a limit of 4 that is higher than the authorized limit (1).

You will have to modify or to delete the defined Limit Range of your project.

##### Memory limits
The same behavior can occur if you have limit ranges defined on memory resources.
The events message will look like `Error creating: pods "neoload-web-1-6rzzk" is forbidden: maximum memory usage per Pod is 1Gi, but limit is 5368709120`.

###### Example
```shell
user@host:~$ oc get events
LAST SEEN   FIRST SEEN   COUNT     NAME                                    KIND                   SUBOBJECT                     TYPE      REASON              SOURCE                                        MESSAGE
3s          3s           1         neoload-web-1-deploy.1603cee243d39112   Pod                                                  Normal    Scheduled           default-scheduler                             Successfully assigned my-workspace/neoload-web-1-deploy to openshift.mycompany.com
3s          3s           1         neoload-web.1603cee23c2c6eea            DeploymentConfig                                     Normal    DeploymentCreated   deploymentconfig-controller                   Created new replication controller "neoload-web-1" for version 1
0s          0s           1         neoload-web-1-deploy.1603cee32275da51   Pod                    spec.containers{deployment}   Normal    Pulled              kubelet, openshift.mycompany.com              Container image "docker.io/openshift/origin-deployer:v3.11.0" already present on machine
0s          0s           1         neoload-web-1-deploy.1603cee359360499   Pod                    spec.containers{deployment}   Normal    Created             kubelet, openshift.mycompany.com              Created container
0s          0s           1         neoload-web-1-deploy.1603cee376df0c15   Pod                    spec.containers{deployment}   Normal    Started             kubelet, openshift.mycompany.com              Started container
0s          0s           1         neoload-web-1.1603cee390a2d2bc          ReplicationController                                Warning   FailedCreate        replication-controller                        Error creating: pods "neoload-web-1-zkkj7" is forbidden: maximum memory usage per Pod is 1Gi, but limit is 5368709120.
0s          0s           1         neoload-web-1.1603cee3945cf7d0          ReplicationController                                Warning   FailedCreate        replication-controller                        Error creating: pods "neoload-web-1-9qr9f" is forbidden: maximum memory usage per Pod is 1Gi, but limit is 5368709120.
0s          0s           1         neoload-web-1.1603cee39508bf39          ReplicationController                                Warning   FailedCreate        replication-controller                        Error creating: pods "neoload-web-1-6rzzk" is forbidden: maximum memory usage per Pod is 1Gi, but limit is 5368709120.
0s          0s           1         neoload-web-1.1603cee3956f93f1          ReplicationController                                Warning   FailedCreate        replication-controller                        Error creating: pods "neoload-web-1-6f7qb" is forbidden: maximum memory usage per Pod is 1Gi, but limit is 5368709120.
```

#### Quotas
Openshift allows defining per project [Quotas](https://docs.openshift.com/container-platform/3.11/dev_guide/compute_resources.html#dev-quotas) on resources usage.

If the defined Quota of your project does not allow deploying NeoLoad Web, then the same kind of behavior as for CPU and memory limits will occur.

In this case the event message will look like `Error creating: pods "neoload-web-1-jg6l2" is forbidden: exceeded quota: my-quota, requested: requests.cpu=2, used: requests.cpu=100m, limited: requests.cpu=1`.


You can check if you project is subject to quotas using the command `oc get quotas`.
```shell
user@host:~$ oc get quota
NAME       CREATED AT
my-quota   2020-04-08T10:04:21Z

user@host:~$ oc describe quota my-quota
Name:         my-quota
Namespace:    my-workspace
Resource      Used  Hard
--------      ----  ----
requests.cpu  100m  1
```




### Image pull back off error

Deployment has been succesfully done but only `neoload-web-1-deploy` Pod is running and `neoload-web-1-xxxxx` is in `ImagePullBackOff` status.
#### Ouput example
```shell
user@host:~$ oc process -f ./neoload-web-template.yaml -p NLW_VERSION=7.0.0 -p MONGODB_URL=mongo.mycompany.com:27017/admin -p SECRET_KEY=azertyui -p OPENSHIFT_ROUTE_HOSTNAME=openshift.mycompany.com | oc apply -f -deploymentconfig.apps.openshift.io/neoload-web created
service/neoload-api-svc created
route.route.openshift.io/neoload-api created
service/neoload-files-svc created
route.route.openshift.io/neoload-files created
service/neoload-front-svc created
route.route.openshift.io/neoload-front created
service/neoload-front-admin-svc created
route.route.openshift.io/neoload-front-admin created
service/neoload-back-admin-svc created
route.route.openshift.io/neoload-back-admin created

user@host:~$ oc get dc
NAME          REVISION   DESIRED   CURRENT   TRIGGERED BY
neoload-web   1          1         1         config

user@host:~$ oc get pods
NAME                   READY     STATUS             RESTARTS   AGE
neoload-web-1-deploy   1/1       Running            0          45s
neoload-web-1-p9lbt    0/2       ImagePullBackOff   0          37s

user@host:~$ oc get events
LAST SEEN   FIRST SEEN   COUNT     NAME                                    KIND                    SUBOBJECT                               TYPE      REASON              SOURCE                                       MESSAGE
34s         34s          1         neoload-web-1-deploy.160417a5435e1f31   Pod                                                             Normal    Scheduled           default-scheduler                            Successfully assigned my-workspace/neoload-web-1-deploy to openshift.mycompany.com
34s         34s          1         neoload-web.160417a53eb01264            DeploymentConfig                                                Normal    DeploymentCreated   deploymentconfig-controller                  Created new replication controller "neoload-web-1" for version 1
31s         31s          1         neoload-web-1-deploy.160417a60d0b1e1d   Pod                     spec.containers{deployment}             Normal    Pulled              kubelet, openshift.mycompany.com             Container image "docker.io/openshift/origin-deployer:v3.11.0" already present on machine
30s         30s          1         neoload-web-1-deploy.160417a631d0a1c7   Pod                     spec.containers{deployment}             Normal    Created             kubelet, openshift.mycompany.com             Created container
30s         30s          1         neoload-web-1-deploy.160417a646413a2c   Pod                     spec.containers{deployment}             Normal    Started             kubelet, openshift.mycompany.com             Started container
26s         26s          1         neoload-web-1.160417a737d4e30d          ReplicationController                                           Normal    SuccessfulCreate    replication-controller                       Created pod: neoload-web-1-p9lbt
26s         26s          1         neoload-web-1-p9lbt.160417a73b588731    Pod                                                             Normal    Scheduled           default-scheduler                            Successfully assigned my-workspace/neoload-web-1-p9lbt to rdopenshift02
14s         17s          2         neoload-web-1-p9lbt.160417a9517d59fe    Pod                     spec.containers{neoload-web-frontend}   Normal    BackOff             kubelet, rdopenshift02                       Back-off pulling image "neotys/neoload-web-frontend:7.0.0"
14s         17s          2         neoload-web-1-p9lbt.160417a9517d82f6    Pod                     spec.containers{neoload-web-frontend}   Warning   Failed              kubelet, rdopenshift02                       Error: ImagePullBackOff
14s         17s          2         neoload-web-1-p9lbt.160417a95174a57b    Pod                     spec.containers{neoload-web-backend}    Warning   Failed              kubelet, rdopenshift02                       Error: ImagePullBackOff
11s         23s          2         neoload-web-1-p9lbt.160417a7e85677fa    Pod                     spec.containers{neoload-web-backend}    Normal    Pulling             kubelet, rdopenshift02                       pulling image "neotys/neoload-web-backend:7.0.0"
9s          22s          2         neoload-web-1-p9lbt.160417a8330fee0c    Pod                     spec.containers{neoload-web-frontend}   Normal    Pulling             kubelet, rdopenshift02                       pulling image "neotys/neoload-web-frontend:7.0.0"
9s          22s          2         neoload-web-1-p9lbt.160417a83302858c    Pod                     spec.containers{neoload-web-backend}    Warning   Failed              kubelet, rdopenshift02                       Failed to pull image "neotys/neoload-web-backend:7.0.0": rpc error: code = Unknown desc = manifest for docker.io/neotys/neoload-web-backend:7.0.0 not found
9s          22s          2         neoload-web-1-p9lbt.160417a83302ecd1    Pod                     spec.containers{neoload-web-backend}    Warning   Failed              kubelet, rdopenshift02                       Error: ErrImagePull
8s          20s          2         neoload-web-1-p9lbt.160417a87ae5f6bc    Pod                     spec.containers{neoload-web-frontend}   Warning   Failed              kubelet, rdopenshift02                       Error: ErrImagePull
8s          19s          4         neoload-web-1-p9lbt.160417a8af5c2ebf    Pod                                                             Normal    SandboxChanged      kubelet, rdopenshift02                       Pod sandbox changed, it will be killed and re-created.
8s          20s          2         neoload-web-1-p9lbt.160417a87ae5abb7    Pod                     spec.containers{neoload-web-frontend}   Warning   Failed              kubelet, rdopenshift02                       Failed to pull image "neotys/neoload-web-frontend:7.0.0": rpc error: code = Unknown desc = manifest for docker.io/neotys/neoload-web-frontend:7.0.0 not found
5s          17s          3         neoload-web-1-p9lbt.160417a951747cec    Pod                     spec.containers{neoload-web-backend}    Normal    BackOff             kubelet, rdopenshift02                       Back-off pulling image "neotys/neoload-web-backend:7.0.0"
```

#### Possible causes
##### Not existing version
You will face this kind of error if you specify a bad `NLW_VERSION`. In the example, the version is 7.0.0.
The version are mapped to docker image tag, and since the tag doesn't exist OpenShift is not able to pull the corresponding Docker image.
The event message is clear about that: `Failed to pull image "neotys/neoload-web-frontend:7.0.0": rpc error: code = Unknown desc = manifest for docker.io/neotys/neoload-web-frontend:7.0.0 not found`.

##### OpenShift Image Policy
[Openshift Image Policy](https://docs.openshift.com/container-platform/3.11/admin_guide/image_policy.html#configuring-registries-allowed-for-import) can also be the cause of this kind of behavior.
Depending on your defined cluster Image Policy, it is possible that OpenShift is not allowed to pull images from [Docker Hub](https://www.docker.com/products/docker-hub).


## License

NeoLoad Web is licensed under the following  [License Agreement](http://www.neotys.com/documents/legal/eula/neoload/eula_en.html). You must agree to this license agreement to download and use NeoLoad Web.

Note: This license do not permit further distribution.

## User Feedback

For general issues relating to NeoLoad you can get help from  [Neotys Support](https://www.neotys.com/community/?from=%2Faccountarea%2Fcasecreate.php)  or  [Neotys Community](http://answers.neotys.com/).
