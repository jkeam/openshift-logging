# Installation

1.  `oc apply -f ./02_installation/02_01_installation.yaml`

2.  `oc get csv --all-namespaces` - Verify `4.6.0-202106010807.p0.git.c07c7ab` as `Succeeded` in every namespace

3.  `oc apply -f ./02_installation/02_02_installation.yaml`

4.  `oc get csv -n openshift-logging` - Verify `clusterlogging.4.6.0-202106010807.p0.git.e091c13` as `Succeeded`

5.  `oc apply -f ./02_installation/02_03_installation.yaml` - To customize the logging instance, update before applying

6.  `oc get pods -n openshift-logging` - Verify installation, you will see something like the following

    ```shell
    NAME                                            READY   STATUS    RESTARTS   AGE
    cluster-logging-operator-66f77ffccb-ppzbg       1/1     Running   0          7m
    elasticsearch-cdm-ftuhduuw-1-ffc4b9566-q6bhp    2/2     Running   0          2m40s
    elasticsearch-cdm-ftuhduuw-2-7b4994dbfc-rd2gc   2/2     Running   0          2m36s
    elasticsearch-cdm-ftuhduuw-3-84b5ff7ff8-gqnm2   2/2     Running   0          2m4s
    fluentd-587vb                                   1/1     Running   0          2m26s
    fluentd-7mpb9                                   1/1     Running   0          2m30s
    fluentd-flm6j                                   1/1     Running   0          2m33s
    fluentd-gn4rn                                   1/1     Running   0          2m26s
    fluentd-nlgb6                                   1/1     Running   0          2m30s
    fluentd-snpkt                                   1/1     Running   0          2m28s
    kibana-d6d5668c5-rppqm                          2/2     Running   0          2m39s
    ```

7.  Optionally, you can configure OCP Events to be collected by Fluentd, but keep in mind this adds additional load to Fluentd and can impact the number of other log messages that can be processed.  [Instructions to set this up are here](https://docs.openshift.com/container-platform/4.7/logging/cluster-logging-eventrouter.html#cluster-logging-eventrouter).

    ```shell
    oc apply -f ./02_installation/02_05_installation.yaml
    ```

8.  Optionally, you can configure OCP audit logs to be collected by Fluentd.  Keep in mind that the internally OCP Elasticsearch is not encrypted so neither will this log info, which is why this is disabled by default.  [Instructions to set this up are here](https://docs.openshift.com/container-platform/4.7/logging/config/cluster-logging-log-store.html#cluster-logging-elasticsearch-audit_cluster-logging-store).

    ```shell
    oc apply -f ./02_installation/02_04_installation.yaml
    ```

    Or for an existing one `oc edit clusterlogforwarder -n openshift-logging`.  Make sure `pipelines` has `audit`.

    ```shell
    apiVersion: "logging.openshift.io/v1"
    kind: ClusterLogForwarder
    metadata:
      name: instance
      namespace: openshift-logging
    spec:
      outputs:
       - name: elasticsearch-insecure
         type: "elasticsearch"
         url: http://elasticsearch-insecure.messaging.svc.cluster.local
         insecure: true
       - name: elasticsearch-secure
         type: "elasticsearch"
         url: https://elasticsearch-secure.messaging.svc.cluster.local
         secret:
           name: es-audit
       - name: secureforward-offcluster
         type: "fluentdForward"
         url: https://secureforward.offcluster.com:24224
         secret:
           name: secureforward
      pipelines:
       - name: container-logs
         inputRefs:
         - application
         outputRefs:
         - secureforward-offcluster
       - name: infra-logs
         inputRefs:
         - infrastructure
         outputRefs:
         - elasticsearch-insecure
       - name: audit-logs
         inputRefs:
         - audit
         outputRefs:
         - elasticsearch-secure
         - default
    ```

9.  Optionally, if you configured multitenant isolation mode with the OpenShift SDN CNI plug-in set to `Multitenant` or `NetworkPolicy` mode, use the following to join the two projects.  [Details can be found here](https://docs.openshift.com/container-platform/4.7/logging/cluster-logging-deploying.html#cluster-logging-deploy-multitenant_cluster-logging-deploying).

    ```shell
    # Multitenant Mode
    oc adm pod-network join-projects --to=openshift-operators-redhat openshift-logging
    ```

    ```shell
    # NetworkPolicy Mode
    kind: NetworkPolicy
    apiVersion: networking.k8s.io/v1
    metadata:
      name: allow-openshift-operators-redhat
    spec:
      ingress:
        - from:
          - namespaceSelector:
              matchLabels:
                project: openshift-operators-redhat
    ```
