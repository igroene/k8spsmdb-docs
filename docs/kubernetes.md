# Install Percona server for MongoDB on Kubernetes


1. Clone the percona-server-mongodb-operator repository:

```bash
$ git clone -b v1.12.0 https://github.com/percona/percona-server-mongodb-operator
cd percona-server-mongodb-operator
```

**NOTE**: It is crucial to specify the right branch with `-b`
option while cloning the code on this step. Please be careful.


2. The Custom Resource Definition for Percona Server for MongoDB should be
created from the `deploy/crd.yaml` file. The Custom Resource Definition
extends the standard set of resources which Kubernetes “knows” about with the
new items, in our case these items are the core of the operator. [Apply it](https://kubernetes.io/docs/reference/using-api/server-side-apply/) as follows:

```bash
$ kubectl apply --server-side -f deploy/crd.yaml
```

This step should be done only once; the step does not need to be repeated
with any other Operator deployments.


3. Create a namespace and set the context for the namespace. The resource names
must be unique within the namespace and provide a way to divide cluster
resources between users spread across multiple projects.

So, create the namespace and save it in the namespace context for subsequent
commands as follows (replace the `<namespace name>` placeholder with some
descriptive name):

```bash
$ kubectl create namespace <namespace name>
$ kubectl config set-context $(kubectl config current-context) --namespace=<namespace name>
```

At success, you will see the message that namespace/<namespace name> was
created, and the context was modified.


4. The role-based access control (RBAC) for Percona Server for MongoDB is
configured with the `deploy/rbac.yaml` file. Role-based access is based on
defined roles and the available actions which correspond to each role. The
role and actions are defined for Kubernetes resources in the yaml file.
Further details about users and roles can be found in [Kubernetes documentation](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#default-roles-and-role-bindings).

```bash
$ kubectl apply -f deploy/rbac.yaml
```

**NOTE**: Setting RBAC requires your user to have cluster-admin role
privileges. For example, those using Google Kubernetes Engine can
grant user needed privileges with the following command:

```default
$ kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value core/account)
```


5. Start the operator within Kubernetes:

```bash
$ kubectl apply -f deploy/operator.yaml
```


6. Add the MongoDB Users secrets to Kubernetes. These secrets
should be placed as plain text in the stringData section of the
`deploy/secrets.yaml` file as login name and
passwords for the user accounts (see [Kubernetes
documentation](https://kubernetes.io/docs/concepts/configuration/secret/)
for details).

After editing the yaml file, MongoDB Users secrets should be created
using the following command:

```bash
$ kubectl create -f deploy/secrets.yaml
```

More details about secrets can be found in [Users](users.md#users).


7. Now certificates should be generated. By default, the Operator generates
certificates automatically, and no actions are required at this step. Still,
you can generate and apply your own certificates as secrets according
to the [TLS instructions](TLS.md#tls).


8. After the operator is started, Percona Server for MongoDB cluster can
be created with the following command:

```bash
$ kubectl apply -f deploy/cr.yaml
```

The creation process may take some time. The process is over when all Pods
have reached their Running status. You can check it with the following command:

```bash
$ kubectl get pods
```

The result should look as follows:

```text
NAME                                               READY   STATUS    RESTARTS   AGE
my-cluster-name-cfg-0                              2/2     Running   0          11m
my-cluster-name-cfg-1                              2/2     Running   1          10m
my-cluster-name-cfg-2                              2/2     Running   1          9m
my-cluster-name-mongos-55659468f7-2kvc2            1/1     Running   0          11m
my-cluster-name-mongos-55659468f7-7jfqc            1/1     Running   0          11m
my-cluster-name-mongos-55659468f7-dfwcj            1/1     Running   0          11m
my-cluster-name-rs0-0                              2/2     Running   0          11m
my-cluster-name-rs0-1                              2/2     Running   0          10m
my-cluster-name-rs0-2                              2/2     Running   0          9m
percona-server-mongodb-operator-6fc78d686d-26hdz   1/1     Running   0          37m
```


9. Check connectivity to newly created cluster, using the login (which is
`userAdmin`) and corresponding password from the secret:

```bash
$ kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:4.4.13-13 --restart=Never -- bash -il
percona-client:/$ mongo "mongodb://userAdmin:userAdmin123456@my-cluster-name-mongos.<namespace name>.svc.cluster.local/admin?ssl=false"
```