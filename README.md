rabbitmq-vagrant-dual-node
==========================

Vagrant configuration for setting up a single server dual node rabbitmq cluster and a default HA policy. Useful for testing client failover scenarios.

## Usage

All queues with a name prefixed with "ha." will have the high availability policy applied.
