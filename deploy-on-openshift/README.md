
# Deploy OpenSearch Cluster Operator via Helm charts on OpenShift



Install Helm CLI for OpenShift

```bash
curl -L https://mirror.openshift.com/pub/openshift-v4/clients/helm/latest/helm-linux-amd64 -o /usr/local/bin/helm
chmod +x /usr/local/bin/helm
```

Add opensearch-operator Helm Charts

```bash
helm repo add opensearch-operator https://opensearch-project.github.io/opensearch-k8s-operator/
```


Create new project(namespace) for OpenSearch

```bash
oc new-project opensearch
```

Git clone
```bash
git clone https://github.com/rh-bjeon/opensearch-k8s-operator && cd deploy-on-openshift
```


Install `opensearch-operator` Helm charts on new Project with customized values.yaml

```bash
helm install opensearch-operator opensearch-operator/opensearch-operator -f values.yaml -n opensearch
```
> ```
> ### Install Helm chart using a values.yaml file with the following values ​​added:
> manager:
> extraEnv:
>     - name: SKIP_INIT_CONTAINER
>       value: "true"
> ```


Label the worker nodes on which you will deploy the OpenSearch Cluster.
```bash
oc label nodes <WorkerNode1> type=opensearch 
oc label nodes <WorkerNode2> type=opensearch 
oc label nodes <WorkerNode3> type=opensearch 
```

For OpenShift, you need to create a node tuning operator openshift-tuned.yaml, to set the vm.map_max_count to 262144 on the labeled worker nodes.
Run the following command to apply the openshift-tuned.yaml to the labeled worker nodes to set the vm.map_max_count to 262144.
(Ref: https://docs.microfocus.com/doc/115/25.3/402-setupopensearchoperator#Create_the_values_yaml_for_the_Operator)
```bash
oc apply -f openshift-tuned.yaml
```


And check the opensearch-cluster.yaml file.
- Metadata.name and metadata.namespace can be modified, but are not necessary.
- You should modify spec.nodePools.persistence.pvc.storageClass to suit your environment. This example uses an AWS environment.

- To expose Dashboards outside of the cluster, this example uses Operator-generated certificates internally and let an OpenShift Router present a valid certificate from an accredited CA.
- If you want to use your own certificate, you need to provide it as a Kubernetes TLS secret. Please check the contents of this link(https://github.com/opensearch-project/opensearch-k8s-operator/blob/main/docs/userguide/main.md#dashboards-http).

- For the commented "OPTIONAL" items, please refer to the official user guide.


Once you've completed editing the opensearch-cluster.yaml file, apply it to your cluster.
```bash
oc apply -f opensearch-cluster.yaml
```

