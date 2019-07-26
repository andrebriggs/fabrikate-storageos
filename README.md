# Fabrikate StorageOS

Uses the reccomended StorageOS chart from https://github.com/storageos/charts/tree/master/stable/storageos-operator

Installs the StorageOS Operator, and then installs a StorageOS cluster.

Kubernetes ships with a StorageOS Kubernetes [volume plugin](https://kubernetes.io/docs/concepts/storage/storage-classes/#storageos) since it is a storage class a [provisioner](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner).

You can see the storageclasses your application's StatefulSet is using by calling 
```
$ kubectl get storageclass
```
If you see a StorageOS provisioned storageclass then your statefulset is configured.

## Uninstalling StorageOS
```
$ kubectl delete all --all -n storageos
$ kubectl delete all --all -n storageos-operator
```

# Manual Kafka/Zookeeper End to End Example

## Install Kafka/Zookeeper
1. Add repo to helm: `$ helm repo add incubator http://storage.googleapis.com/kubernetes-charts-incubator`
2. Install a release named **kafka**: `$ helm install --name kafka incubator/kafka`
3. Set the Kafka StatefulSet to use a storageclass that StorageOS supports (see Statefulset defintion [here](https://docs.storageos.com/docs/usecases/kubernetes/kafka))

**Note**: If you have issues with Tiller RBAC run (via [link](https://github.com/helm/helm/issues/3130#issuecomment-372931407))
```
kubectl --namespace kube-system create serviceaccount tiller

kubectl create clusterrolebinding tiller-cluster-rule \
 --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

kubectl --namespace kube-system patch deploy tiller-deploy \
 -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}' 
 ```

## Uninstalling Kakfa via Helm
```
$ helm list
NAME              	REVISION	UPDATED                 	STATUS  	CHART                          	APP VERSION	NAMESPACE         
kafka             	1       	Sun Jun 30 22:15:19 2019	DEPLOYED	kafka-0.16.2                   	5.0.1      	default           
$ helm delete kafka
release "kafka" deleted
```

## Create Topic
1. List topics: `$ kubectl -n default exec -ti testclient -- ./bin/kafka-topics.sh --zookeeper kafka-zookeeper:2181 --list`
    a. There should be none initially
2. Create a topic: `$ kubectl -n default exec -ti testclient -- ./bin/kafka-topics.sh --zookeeper kafka-zookeeper:2181 --create --topic test-rep-one --partitions 6 --replication-factor 1`

## Run Tests
1. Manually install test client: `$ kubectl apply -f /Users/andrebriggs/Code/kafka-test-client-pod.yaml`
2. Send data to Kafka Cluster that whole persistent volume is backed by StorageOS: `$ kubectl -n default exec -ti testclient -- ./bin/kafka-run-class.sh org.apache.kafka.tools.ProducerPerformance --topic test-rep-one --num-records 5000 --record-size 100 --throughput -1 --print-metrics --producer-props acks=1 bootstrap.servers=kafka:9092 buffer.memory=67108864 batch.size=8196`

## Test Results From Client
```
Metric Name                                                                           Value
kafka-metrics-count:count:{client-id=producer-1}                                    : 76.000
producer-metrics:batch-size-avg:{client-id=producer-1}                              : 3490.667
producer-metrics:batch-size-max:{client-id=producer-1}                              : 8137.000
producer-metrics:batch-split-rate:{client-id=producer-1}                            : 0.000
producer-metrics:buffer-available-bytes:{client-id=producer-1}                      : 67108864.000
producer-metrics:buffer-exhausted-rate:{client-id=producer-1}                       : 0.000
producer-metrics:buffer-total-bytes:{client-id=producer-1}                          : 67108864.000
producer-metrics:bufferpool-wait-ratio:{client-id=producer-1}                       : 0.000
producer-metrics:compression-rate-avg:{client-id=producer-1}                        : 1.000
producer-metrics:connection-close-rate:{client-id=producer-1}                       : 0.000
producer-metrics:connection-count:{client-id=producer-1}                            : 4.000
producer-metrics:connection-creation-rate:{client-id=producer-1}                    : 0.132
producer-metrics:incoming-byte-rate:{client-id=producer-1}                          : 239.947
producer-metrics:io-ratio:{client-id=producer-1}                                    : 0.001
producer-metrics:io-time-ns-avg:{client-id=producer-1}                              : 422965.899
producer-metrics:io-wait-ratio:{client-id=producer-1}                               : 0.004
producer-metrics:io-wait-time-ns-avg:{client-id=producer-1}                         : 1624404.127
producer-metrics:metadata-age:{client-id=producer-1}                                : 0.250
producer-metrics:network-io-rate:{client-id=producer-1}                             : 5.606
producer-metrics:outgoing-byte-rate:{client-id=producer-1}                          : 18491.855
producer-metrics:produce-throttle-time-avg:{client-id=producer-1}                   : 0.000
producer-metrics:produce-throttle-time-max:{client-id=producer-1}                   : 0.000
producer-metrics:record-error-rate:{client-id=producer-1}                           : 0.000
producer-metrics:record-queue-time-avg:{client-id=producer-1}                       : 5.799
producer-metrics:record-queue-time-max:{client-id=producer-1}                       : 41.000
producer-metrics:record-retry-rate:{client-id=producer-1}                           : 0.000
producer-metrics:record-send-rate:{client-id=producer-1}                            : 165.420
producer-metrics:record-size-avg:{client-id=producer-1}                             : 186.000
producer-metrics:record-size-max:{client-id=producer-1}                             : 186.000
producer-metrics:records-per-request-avg:{client-id=producer-1}                     : 62.500
producer-metrics:request-latency-avg:{client-id=producer-1}                         : 19.650
producer-metrics:request-latency-max:{client-id=producer-1}                         : 76.000
producer-metrics:request-rate:{client-id=producer-1}                                : 2.803
producer-metrics:request-size-avg:{client-id=producer-1}                            : 6597.024
producer-metrics:request-size-max:{client-id=producer-1}                            : 16344.000
producer-metrics:requests-in-flight:{client-id=producer-1}                          : 0.000
producer-metrics:response-rate:{client-id=producer-1}                               : 2.804
producer-metrics:select-rate:{client-id=producer-1}                                 : 2.601
producer-metrics:waiting-threads:{client-id=producer-1}                             : 0.000
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node--1}    : 18.473
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-0}     : 75.093
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-1}     : 72.660
producer-node-metrics:incoming-byte-rate:{client-id=producer-1, node-id=node-2}     : 74.380
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node--1}    : 2.209
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-0}     : 6190.030
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-1}     : 6176.315
producer-node-metrics:outgoing-byte-rate:{client-id=producer-1, node-id=node-2}     : 6180.384
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node--1}   : 0.000
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-0}    : 26.741
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-1}    : 17.654
producer-node-metrics:request-latency-avg:{client-id=producer-1, node-id=node-2}    : 14.481
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node--1}   : -Infinity
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-0}    : 76.000
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-1}    : 55.000
producer-node-metrics:request-latency-max:{client-id=producer-1, node-id=node-2}    : 43.000
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node--1}          : 0.066
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-0}           : 0.926
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-1}           : 0.893
producer-node-metrics:request-rate:{client-id=producer-1, node-id=node-2}           : 0.926
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node--1}      : 33.500
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-0}       : 6683.464
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-1}       : 6915.185
producer-node-metrics:request-size-avg:{client-id=producer-1, node-id=node-2}       : 6672.607
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node--1}      : 43.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-0}       : 16344.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-1}       : 16344.000
producer-node-metrics:request-size-max:{client-id=producer-1, node-id=node-2}       : 16344.000
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node--1}         : 0.066
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-0}          : 0.926
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-1}          : 0.893
producer-node-metrics:response-rate:{client-id=producer-1, node-id=node-2}          : 0.926
producer-topic-metrics:byte-rate:{client-id=producer-1, topic=test-rep-one}         : 18362.205
producer-topic-metrics:compression-rate:{client-id=producer-1, topic=test-rep-one}  : 1.000
producer-topic-metrics:record-error-rate:{client-id=producer-1, topic=test-rep-one} : 0.000
producer-topic-metrics:record-retry-rate:{client-id=producer-1, topic=test-rep-one} : 0.000
producer-topic-metrics:record-send-rate:{client-id=producer-1, topic=test-rep-one}  : 165.420
```
