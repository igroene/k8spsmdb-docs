# Using Replica Set Arbiter nodes and non-voting nodes

Percona Server for MongoDB [replication model :material-arrow-top-right:](https://www.percona.com/blog/2018/05/17/mongodb-replica-set-transport-encryption-part-1/)
is based on elections, when nodes of the Replica Set [choose which node :material-arrow-top-right:](https://docs.mongodb.com/manual/core/replica-set-elections/#replica-set-elections)
becomes the primary node.

The need for elections influences the choice of the number of nodes in the cluster.
Elections are the reason to avoid even number of nodes, and to have at least
three and not more than seven participating nodes.

Still, sometimes there is a contradiction between the number of nodes suitable for
elections and the number of nodes needed to store data. You can solve this
contradiction in two ways:

* Add *Arbiter* nodes, which participate in elections, but do not store data,
* Add *non-voting* nodes, which store data but do not participate in elections.

## Adding Arbiter nodes

Normally, each node stores a complete copy of the data,
but there is also a possibility, to reduce disk IO and space used by the
database, to add an [arbiter node :material-arrow-top-right:](https://docs.mongodb.com/manual/core/replica-set-arbiter/).
An arbiter cannot become a primary and does not have a complete copy of the
data. The arbiter does have one election vote and can be the odd number for
elections. The arbiter does not demand a persistent volume.

Percona Operator for MongoDB has the ability to create Replica Set Arbiter nodes
if needed. This feature can be configured in the Replica Set section of the
[deploy/cr.yaml :material-arrow-top-right:](https://github.com/percona/percona-server-mongodb-operator/blob/main/deploy/cr.yaml)
file:

* set `arbiter.enabled` option to `true` to allow Arbiter instances,
* use `arbiter.size` option to set the desired amount of Arbiter instances.

For example, the following keys in `deploy/cr.yaml` will create a cluster
with 4 data instances and 1 Arbiter:

```yaml
....
replsets:
  ....
  size: 4
  ....
  arbiter:
    enabled: true
    size: 1
    ....
```

!!! note

    You can find description of other possible options in the
    [replsets.arbiter section](operator.md#replsets-arbiter-enabled) of the
    [Custom Resource options reference](operator.md#operator-custom-resource-options).

## Adding non-voting nodes

[Non-voting member :material-arrow-top-right:](https://docs.mongodb.com/manual/tutorial/configure-a-non-voting-replica-set-member/)
is a Replica Set node which does not participate in the primary election
process. This feature is required to have more than 7 nodes, or if there is a
[node in the edge location :material-arrow-top-right:](https://en.wikipedia.org/wiki/Edge_computing), which
obviously should not participate in the voting process.

!!! note

    Non-voting nodes support has technical preview status and is not recommended
    for production environments.

!!! note

    It is possible to add a non-voting node in the edge location through the
    `externalNodes` option. Please see [cross-site replication documentation](replication.md#operator-replication)
    for details.

Percona Operator for MongoDB has the ability to configure non-voting nodes in
the Replica Set section of the [deploy/cr.yaml :material-arrow-top-right:](https://github.com/percona/percona-server-mongodb-operator/blob/main/deploy/cr.yaml)
file:

* set `nonvoting.enabled` option to `true` to allow non-voting instances,
* use `nonvoting.size` option to set the desired amount of non-voting instances.

For example, the following keys in `deploy/cr.yaml` will create a cluster with
3 data instances and 1 non-voting instance:

```yaml
....
replsets:
  ....
  size: 3
  ....
  nonvoting:
    enabled: true
    size: 1
    ....
```

!!! note

    You can find description of other possible options in the [replsets.nonvoting section](operator.md#replsets-nonvoting-enabled)
    of the [Custom Resource options reference](operator.md#operator-custom-resource-options).
