# Performance and Scale Alerts for OpenShift
Collection of Performance and Scalability alerts based on Prometheus metrics for various components of OpenShift

## Run
Create the PrometheusRule object and observe the alerts on the OpenShift console or at <prometheus_url>/alerts:
```
$ oc create -f <component>/<alert.yml>
```

## Monitored Components

### Etcd

#### Etcd Database Quota
Etcd is a critical component and datastore of Kubernetes and OpenShift clusters, it stores and replicates the cluster state. For large and dense clusters, etcd can suffer from poor performance if the keyspace grows excessively large and exceeds the space quota. Periodic maintenance of etcd including defragmentation needs to be done to free up space in the data store. It is highly recommended that you monitor Prometheus for etcd metrics and defragment it when needed before etcd raises a cluster-wide alarm that puts the cluster into a maintenance mode, which only accepts key reads and deletes. Some of the key metrics to monitor are etcd_server_quota_backend_bytes which is the current quota limit, etcd_mvcc_db_total_size_in_use_in_bytes which indicates the actual database usage after a history compaction, and etcd_debugging_mvcc_db_total_size_in_bytes which shows the database size including free space waiting for defragmentation.

Enabling the alerts in the repository will fire a critical alert when the etcd database quota is 95% full at any given point of time to alert the user to defrag or increase the quota in order to avoid the alarm getting triggered which blocks all the writes to etcd meaning there can't be any new objects created. This is needed to make sure the cluster supports running large number of nodes and objects. It also fires a warning when there is a sudden surge in etcd writes leading to increase in the etcd database quota size at an alarming rate as it is disruptive. It might be because of a rougue process and it's important to alert the admin.

