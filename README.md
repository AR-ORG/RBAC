# User Management using RBAC

## Create Namespace

```
kubectl create namespace development
```

## Create Service Account

Create a file sa.yaml

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: developer
  namespace: development

```

Apply the changes by executing ` kubectl apply -f sa.yaml` 

## Create Role and RoleBindings

Create a file - role.yaml 

```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: development-user-full-access
  namespace: development
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["*"]
  verbs: ["*"]
- apiGroups: ["batch"]
  resources:
  - jobs
  - cronjobs
  verbs: ["*"]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: development-user-view
  namespace: development
subjects:
- kind: ServiceAccount
  name: developer
  namespace: development
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: development-user-full-access


```

Apply the changes with `kubectl create -f role.yaml`

## Retrieve CA certificate and User token

Get the token details by executing - `kubectl describe sa developer -n development`

```
kubectl describe sa developer -n development
Name:                developer
Namespace:           development
Labels:              <none>
Annotations:         kubectl.kubernetes.io/last-applied-configuration:
                       {"apiVersion":"v1","kind":"ServiceAccount","metadata":{"annotations":{},"name":"developer","namespace":"development"}}
Image pull secrets:  <none>
Mountable secrets:   developer-token-7ccrx
Tokens:              developer-token-7ccrx
Events:              <none>
```

The name of the **token key** in this case is `developer-token-7ccrx`

Use the token key to get CA certificate and User token certificate 

```
kubectl get secret developer-token-7ccrx -n development -o "jsonpath={.data.token}" | base64 -d

kubectl get secret developer-token-7ccrx -n development -o  "jsonpath={.data['ca\.crt']}"
```

Store the output for later use 


## Create a Linux dev user 

```
useradd -m dev
```

Switch to the dev user - 

```
sudo -iu dev
```

## Create kubeconfig for dev user 

Create a directory .kube 
```
mkdir .kube
```

Create a file config inside .kube

```
vi .kube/config
```

Add the below details and replace the placeholders

```
apiVersion: v1
kind: Config
preferences: {}

# Define the cluster
clusters:
- cluster:
    certificate-authority-data: PLACE CERTIFICATE HERE
    # You'll need the API endpoint of your Cluster here:
    server: https://YOUR_KUBERNETES_API_ENDPOINT
  name: my-cluster

# Define the user
users:
- name: developer
  user:
    as-user-extra: {}
    client-key-data: PLACE CERTIFICATE HERE
    token: PLACE USER TOKEN HERE

# Define the context: linking a user to a cluster
contexts:
- context:
    cluster: my-cluster
    namespace: development
    user: developer
  name: development

# Define current context
current-context: development

```

Save the file and test the changes - 

```
kubectl get pods 
No resources found in development namespace.

kubectl get ns 
Error from server (Forbidden): namespaces is forbidden: User "system:serviceaccount:development:developer" cannot list resource "namespaces" in API group "" at the cluster scope
```





