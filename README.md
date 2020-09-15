# Onboarding an Openshift cluster to Check Point Cloudguard CSPM (ex.Dome9)


Please go to the CloudGuard asset onboarding page in cloudguard the pick Kubernetes. enter the cluster name, namespace name and your cloudguard API token then click next pick an organization then next the STOP Enter the command below in your openshift cluster:


> IMPORTANT: Please do not use the Helm chart or manual option kubectl comnands as this is for OpenShift which uses oc command to manasge the K8s cluster. Openshift implementation has different values with regards to deployments config paramaters vs traditional K8s. The deployment file and some other commands had to be customized for OpenShift

### Run the following command (only for new namespace)

```
oc create namespace
```

## PLEASE REPLACE with your the namespace name you created!

### Create a CloudGuard token or use an existing one and add to your cluster secrets

```
oc create secret generic dome9-creds --from-literal=username=1b089072-78c1-4d5a-b365-456a9a5f16b2 --from-literal=secret=ustwhdqj4h956e577kr9ewy8 --namespace
```

### Create a configmap to hold the clusterID

```
oc create configmap cp-resource-management-configmap --from-literal=cluster.id=fc6dfe53-a302-4ca7-9053-276511570e6a --namespace
```

### Run the following commands

```
oc create serviceaccount cp-resource-management --namespace
```
### the cloudguard agent uses the user ID 1000 whereas Openshift scc assign a randown UID from a range
### to allow cloudguard to use UID 1000, please create a file uid1000.json containing:

```
{
    "apiVersion": "v1",
    "kind": "SecurityContextConstraints",
    "metadata": {
        "name": "uid1000"
    },
    "requiredDropCapabilities": [
        "KILL",
        "MKNOD",
        "SYS_CHROOT",
        "SETUID",
        "SETGID"
    ],
    "runAsUser": {
        "type": "MustRunAs",
        "uid": "1000"
    },
    "seLinuxContext": {
        "type": "MustRunAs"
    },
    "supplementalGroups": {
        "type": "RunAsAny"
    },
    "fsGroup": {
        "type": "MustRunAs"
    },
    "volumes": [
        "configMap",
        "downwardAPI",
        "emptyDir",
        "persistentVolumeClaim",
        "projected",
        "secret"
    ]
}

```

### Need to create the new SCC, you need to be an administrator.

```
$ oc create -f uid1000.json --as system:admin
securitycontextconstraints "uid1000" created

```
### Set the SCC to be used by the cloudguard service account that we already created 

```
$ oc adm policy add-scc-to-user uid1000 -z cp-resource-management --as system:admin
```

### Run the following commands
```
oc create clusterrole cp-resource-management --verb=get,list --resource=pods,nodes,services,nodes/proxy,networkpolicies.networking.k8s.io,ingresses.extensions,podsecuritypolicies,roles,rolebindings,clusterroles,clusterrolebindings,serviceaccounts,namespaces
```

```
oc create clusterrolebinding cp-resource-management --clusterrole=cp-resource-management --serviceaccount=prod:cp-resource-management
```

### Deploy CloudGuard agent

```
oc create -f https://secure.dome9.com/v2/assets/files/cp-resource-management.yaml
```

then next and wait for agent to be synced

Start running compliance on your OC cluster
