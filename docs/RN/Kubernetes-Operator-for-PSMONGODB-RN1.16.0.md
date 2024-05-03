# *Percona Operator for MongoDB* 1.16.0

* **Date**

    May 08, 2024

* **Installation**

    [Installing Percona Operator for MongoDB](../System-Requirements.md#installation-guidelines)

## Release Highlights

### General availability of Physical Backups

Two releases ago we added experimental support for [Physical Backups and Restores](../backups.md#physical) to significantly reduce Recovery Time Objective ([RTO :octicons-link-external-16:](https://www.percona.com/blog/backups-and-disaster-recovery/#:~:text=Recovery%20time%20objective%20(RTO)%20is,afford%20to%20lose%20after%20recovery).)), especially for big data sets. Until not this feature was a technical preview, but within this release Percona announces the general availability of physical backups and restores for MongoDB with the Operator.

### Automated volume resizing

Kubernetes supports the Persistent Volume expansion as a stable feature since v1.24. Using it with the Operator previously involved manual operations. Now this is automated, and users can resize their PVCs [by just changing the value](../scaling.md#scale-storage) of the `resources.requests.storage` option in the PerconaServerMongoDB custom resource. This feature is in a technical preview stage and is not recommended for production environments.

## New Features

* {{ k8spsmdbjira(1000) }}: Azure Blob Storage [private endpoints](../operator.html#backup-storages-azure-endpointurl) are now supported for backups
* {{ k8spsmdbjira(1055) }}: The `kubectl get psmdb-backup` command now shows Latest restorable time to make it easier to pick a point-in-time recovery target
* {{ k8spsmdbjira(491) }}: It is now possible to specify the [existing cert-manager issuer](../operator.md#tls-issuerconf-name) which should be used by the Operator
* {{ k8spsmdbjira(733) }}: It is now possible to [resize Persistent Volume Claims](../operator.md#automated-scaling-with-volume-expansion-capability) by patching the PerconaServerMongoDB custom resource: change  `persistentVolumeClaim.resources.requests.storage` and let the Operator do the scaling

## Improvements

* {{ k8spsmdbjira(1004) }}: [Exposing replica set with split-horizon DNS](../expose.md#exposing-replica-set-with-split-horizon-dns) allows to specify URIs with non-standard port numbers, which are particularly useful with the NodePort service type
* {{ k8spsmdbjira(1013) }}: MongoDB 7.0 is now supported
* {{ k8spsmdbjira(1015) }}: Information about backup and restore operations is now included in the Operator's logs
* {{ k8spsmdbjira(951) }} and {{ k8spsmdbjira(1021) }}: The Operator now allows setting custom configuration for Percona Backup for MongoDB
* {{ k8spsmdbjira(1029) }}: Mongod is now run in [quiet mode  :octicons-link-external-16:](https://www.mongodb.com/docs/manual/reference/program/mongod/#std-option-mongod.--quiet) to limit the amount of log messages
* {{ k8spsmdbjira(1032) }}: It is now [possible](../operator.md#sharding-mongos-expose-nodeport) to specify nodePort for mongos Service (thanks to Mike Devresse for contribution)
* {{ k8spsmdbjira(1062) }}: The Operator now sets [appProtocol :octicons-link-external-16:](https://kubernetes.io/docs/concepts/services-networking/service/#application-protocol) to `mongo` for Service objects, which is useful for service mesh implementations (thanks to Søren Mathiasen for contribution)
* {{ k8spsmdbjira(732) }}: [Integration of the Operator with OpenLDAP](../ldap.md#using-ldap-over-tls-connection) can now be secured by using TLS connections
* {{ k8spsmdbjira(755) }}: New `allowInvalidCertificates` option allows to [enable or disable](../operator.md#tls-allowinvalidcertificates) bypassing MongoDB Shell checks for the certificates presented by the mongod/mongos instance, useful for self-signed certificates
* {{ k8spsmdbjira(948) }}: ARM  support was added for the Operator image, making it possible to deploy MongoDB cluster on ARM when the database images become available
* : 
* {{ k8spsmdbjira(993) }}: To avoid backup fail on clusters where Percona Backup for MongoDB resync process takes too long, the Operator now checks if there is still a resync operation working with exponentially increasing interval and total wait time until failure equal to 8715 seconds
* {{ k8spsmdbjira(995) }}: The Operator now allows storing key for [backups server-side AWS KMS encryption](../backups-encryption.md) in a Secret configurable with the  `secrets.sse` Custom Resource option

## Bugs Fixed

* {{ k8spsmdbjira(1011) }}: MongoDB Secret Rotation Broken
* {{ k8spsmdbjira(1014) }}: Certificate Rotation brought the Sharded MongoDB cluster down
* {{ k8spsmdbjira(1018) }}: Running MongoDB container fails if the image provides numactl command
* {{ k8spsmdbjira(1024) }}: Fix variable scope for pbm-entry script
* {{ k8spsmdbjira(1030) }}: clusterServiceDNSMode: "External" makes impossble to re-create cluster with kubectl delete + apply cr.yaml
* {{ k8spsmdbjira(1035) }}: Allow empty secretName for backup jobs
* {{ k8spsmdbjira(1036) }}: Restore to new cluster fails
* {{ k8spsmdbjira(1038) }}: Do not delete mongos services if cluster is in pause state
* {{ k8spsmdbjira(1039) }}: pmm agent is not deleted from server inventory on pod termination
* {{ k8spsmdbjira(1053) }}: Restoring a physical backup to different clusters in same namespace fails
* {{ k8spsmdbjira(1056) }}: delete-backup finalizer doesn't work with IAM Roles for Service Accounts
* {{ k8spsmdbjira(1058) }}: Checking authorization failed errors in Mongo Logs
* {{ k8spsmdbjira(1065) }}: Fix pitr-sharded test on EKS
* {{ k8spsmdbjira(1070) }}: Panic in delete-psmdb-pods-in-order finalizer
* {{ k8spsmdbjira(780) }}: Failed to downscale/upscale cluster to unsafe configuration
* {{ k8spsmdbjira(786) }}: When Changing to allowUnsafeConfigurations: true cluster goes to failures and mongos does not get to Ready state
* {{ k8spsmdbjira(865) }}: Arbiter nodeAntiaffinity allows to run it on the same worker as mongo-rs data nodes
* {{ k8spsmdbjira(935) }}: Get backup status command should be changed
* {{ k8spsmdbjira(940) }}: Operator doesn't set serviceAccountName in mongos statefulset
* {{ k8spsmdbjira(979) }}: expose restore variables for PBM (possible OOM/failure with defaults)
* {{ k8spsmdbjira(985) }}: pbmPod key in backup object only shows one replica/pod

## Deprecation and removal



## Supported Platforms

The Operator was developed and tested with Percona Server for MongoDB 4.4.24,
5.0.20, and 6.0.9. Other options may also work but have not been tested. The
Operator also uses Percona Backup for MongoDB 2.3.0.

The following platforms were tested and are officially supported by the Operator
1.15.0:

* [Google Kubernetes Engine (GKE) :octicons-link-external-16:](https://cloud.google.com/kubernetes-engine) 1.24-1.28

* [Amazon Elastic Container Service for Kubernetes (EKS) :octicons-link-external-16:](https://aws.amazon.com) 1.24-1.28

* [OpenShift Container Platform :octicons-link-external-16:](https://www.redhat.com/en/technologies/cloud-computing/openshift) 4.11 - 4.13

* [Azure Kubernetes Service (AKS) :octicons-link-external-16:](https://azure.microsoft.com/en-us/services/kubernetes-service/) 1.25-1.28

* [Minikube :octicons-link-external-16:](https://github.com/kubernetes/minikube) 1.31.2 (based on Kubernetes 1.28)

This list only includes the platforms that the Percona Operators are specifically tested on as part of the release process. Other Kubernetes flavors and versions depend on the backward compatibility offered by Kubernetes itself.