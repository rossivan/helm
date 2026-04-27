# what is helm?
# de-facto standard for distributing software for kubernetes (k8s)
# combination of package manager and templating engine
# primary use-cases: application deployment and environment management

# pieces of helm charts:
# at the top you helm (or oci) repository to store helm charts
# within that will be one or more helm charts
# this is further structued as meta-data as well as templates where kubernetes resource definitions live
# in the meta-data there is a values.yaml file which is the interface with which you can configure and customize your templates
# so, the values in the values.yaml file or if passed through environment variables will get templated in to your templates
# and rendered out as kubernetes manifests

# when we install a chart its going to create an object called a release within our cluster and the release will have
# rendered version of thos templates and deploy them into the kubernetes cluster

# 4 important features that matter:
# how to reference metadata within your templates - you could reference the chart object, release object or the values object
# Variables
# Conditionals
# Loops


# confirm helm is installed
$ helm version (which runs only as a client side tool)
version.BuildInfo{Version:"v3.15.0", GitCommit:"v3.15.0", GitTreeState:"", GoVersion:"go1.22.2"}


# add bitnami helm repository to my set of helm repositories
$ helm repo add bitnami https://charts.bitnami.com/bitnami


# search bitnami repository for PostgreSQL
$ helm search repo bitnami/postgresql --versions


# login to docker OCI registry
$ helm registry login registry-1.docker.io
Username: rossivan@yahoo.co.uk
Password: 
Login Succeeded


# list tags in the OCI registry
# you can't use the helm search command against the OCI repository
$ oras repo tags registry-1.docker.io/bitnamicharts/postgresql
11.9.10
11.9.11
...


# pull PostgreSQL chart from bitnami repository
$ helm pull bitnami/postgresql --version=${POSTGRES_VERSION_2}


# show values for PostgreSQL chart without the need to pull the chart
$ helm show values bitnami/postgresql --version=${POSTGRES_VERSION_2}


# install PostgreSQL chart
$ helm install postgresql bitnami/postgresql \
          --version=${POSTGRES_VERSION_1} \
          --create-namespace \
          --values=values.yaml
NAME: postgresql
LAST DEPLOYED: Sun Apr 19 23:47:21 2026
NAMESPACE: 05--postgresql
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
CHART NAME: postgresql
CHART VERSION: 15.4.1
APP VERSION: 16.3.0

** Please be patient while the chart is being deployed **

PostgreSQL can be accessed via port 5432 on the following DNS names from within your cluster:

    postgresql.05--postgresql.svc.cluster.local - Read/Write connection

To get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace 05--postgresql postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)

To connect to your database run the following command:

    kubectl run postgresql-client --rm --tty -i --restart='Never' --namespace 05--postgresql --image docker.io/bitnami/postgresql:16.3.0-debian-12-r9 --env="PGPASSWORD=$POSTGRES_PASSWORD" \
      --command -- psql --host postgresql -U postgres -d postgres -p 5432

    > NOTE: If you access the container using bash, make sure that you execute "/opt/bitnami/scripts/postgresql/entrypoint.sh /bin/bash" in order to avoid the error "psql: local user with ID 1001} does not exist"

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace 05--postgresql svc/postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432

WARNING: The configured password will be ignored on new installation in case when previous PostgreSQL release was deleted through the helm command. In that case, old PVC will have an old password, and setting it through helm won't take effect. Deleting persistent volumes (PVs) will solve the issue.

WARNING: There are "resources" sections in the chart not set. Using "resourcesPreset" is not recommended for production. For production installations, please set the following values according to your workload needs:
  - primary.resources
  - readReplicas.resources
+info https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/


$ kubectl get all
NAME               READY   STATUS             RESTARTS   AGE
pod/postgresql-0   0/1     ImagePullBackOff   0          4m24s

NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/postgresql      ClusterIP   10.96.160.50   <none>        5432/TCP   4m24s
service/postgresql-hl   ClusterIP   None           <none>        5432/TCP   4m24s

NAME                          READY   AGE
statefulset.apps/postgresql   0/1     4m24s


# the pod can’t download its container image, so it never even gets to start
$ kubectl describe pod postgresql-0
...
Warning  Failed     2m39s (x4 over 3m56s)  kubelet            Failed to pull image "docker.io/bitnami/postgresql:16.3.0-debian-12-r9": rpc error: code = NotFound desc = failed to pull and unpack image "docker.io/bitnami/postgresql:16.3.0-debian-12-r9": failed to resolve reference "docker.io/bitnami/postgresql:16.3.0-debian-12-r9": docker.io/bitnami/postgresql:16.3.0-debian-12-r9: not found
...


# list helm releases
$ helm list -n ${NAMESPACE}
NAME            NAMESPACE       REVISION        UPDATED                                 STATUS          CHART                   APP VERSION
postgresql      05--postgresql  1               2026-04-20 20:09:41.085389976 -0400 EDT deployed        postgresql-15.4.1       16.3.0     


# get values for PostgreSQL release
$ helm get values postgresql
USER-SUPPLIED VALUES:
commonAnnotations:
  foo: bar


# Get manifests for PostgreSQL release
$ helm get manifest postgresql


# helm environment variables
$ helm env
HELM_BIN="helm"
HELM_BURST_LIMIT="100"
HELM_CACHE_HOME="/home/rossi/.cache/helm"
HELM_CONFIG_HOME="/home/rossi/.config/helm"
HELM_DATA_HOME="/home/rossi/.local/share/helm"
HELM_DEBUG="false"
HELM_KUBEAPISERVER=""
HELM_KUBEASGROUPS=""
HELM_KUBEASUSER=""
HELM_KUBECAFILE=""
HELM_KUBECONTEXT=""
HELM_KUBEINSECURE_SKIP_TLS_VERIFY="false"
HELM_KUBETLS_SERVER_NAME=""
HELM_KUBETOKEN=""
HELM_MAX_HISTORY="10"
HELM_NAMESPACE="05--postgresql"
HELM_PLUGINS="/home/rossi/.local/share/helm/plugins"
HELM_QPS="0.00"
HELM_REGISTRY_CONFIG="/home/rossi/.config/helm/registry/config.json"
HELM_REPOSITORY_CACHE="/home/rossi/.cache/helm/repository"
HELM_REPOSITORY_CONFIG="/home/rossi/.config/helm/repositories.yaml"


# list helm repos
$ helm repo list
NAME    URL                               
bitnami https://charts.bitnami.com/bitnami


# get values for PostgreSQL release
$ helm get values postgresql


# uninstall PostgreSQL release
$ helm uninstall postgresql 


================================

Chart.yaml
==========
apiVersion: v2
name: minimal
description: A tiny helm chart for educational purposes
type: application
version: 0.1.0
appVersion: 1.26.0


values.yaml
===========
environment: production

configData:
  - key: conditionalKey
    value: "this will be included"
    enabled: true


configmap.yaml
==============
apiVersion: v1
kind: ConfigMap
metadata:
  name: helm-{{ .Release.Name }}-configmap
data:
  appVersion: {{ .Chart.AppVersion }}
  environment: {{ .Values.environment | quote }}
  envShort: {{ template "envShort" . }} 
  {{- range .Values.configData }}
  {{- if .enabled }}
  {{ .key }}: {{ .value | quote }}
  {{- end }}
  {{- end }}


_helpers.tpl
============
{{- define "envShort" -}}
{{- if eq .Values.environment "production" -}}
prod
{{- else -}}
non-prod
{{- end -}}
{{- end -}}




# the release name is 'minimal' i.e. ".Release.Name"
$ helm upgrade --install minimal ./minimal
Release "minimal" does not exist. Installing it now.
NAME: minimal
LAST DEPLOYED: Mon Apr 20 21:42:08 2026
NAMESPACE: 05--charts
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
Hi 👋 -- you just deployed prod


$ kubectl get cm
NAME                     DATA   AGE
helm-minimal-configmap   4      9m58s
kube-root-ca.crt         1      14m

$ kubectl get cm helm-minimal-configmap -o yaml | yq
apiVersion: v1
data:
  appVersion: 1.26.0
  conditionalKey: this will be included
  envShort: prod
  environment: production
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: minimal
    meta.helm.sh/release-namespace: 05--charts
  creationTimestamp: "2026-04-21T01:42:08Z"
  labels:
    app.kubernetes.io/managed-by: Helm
  name: helm-minimal-configmap
  namespace: 05--charts
  resourceVersion: "1996"
  uid: 6aba6da8-4b46-4083-96c5-26d46ee02433


# NOW CHANGING VALUES.YAML FILE, REST REMAINS SAME


# values-alt.yaml
=================
environment: staging

configData:
  - key: conditionalKey
    value: "this wont be included"
    enabled: false


$ helm upgrade --install minimal ./minimal --values=./minimal/values-alt.yaml
Release "minimal" has been upgraded. Happy Helming!
NAME: minimal
LAST DEPLOYED: Mon Apr 20 21:58:56 2026
NAMESPACE: 05--charts
STATUS: deployed
REVISION: 2
TEST SUITE: None
NOTES:
Hi 👋 -- you just deployed non-prod


$ kubectl get cm
NAME                     DATA   AGE
helm-minimal-configmap   3      17m
kube-root-ca.crt         1      21m


$ kubectl get cm helm-minimal-configmap -o yaml | yq
apiVersion: v1
data:
  appVersion: 1.26.0
  envShort: non-prod
  environment: staging
kind: ConfigMap
metadata:
  annotations:
    meta.helm.sh/release-name: minimal
    meta.helm.sh/release-namespace: 05--charts
  creationTimestamp: "2026-04-21T01:42:08Z"
  labels:
    app.kubernetes.io/managed-by: Helm
  name: helm-minimal-configmap
  namespace: 05--charts
  resourceVersion: "3650"
  uid: 6aba6da8-4b46-4083-96c5-26d46ee02433