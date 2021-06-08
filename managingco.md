

```

VER=$(oc get clusterversion version -o jsonpath='{.status.desired.image}')

oc adm release extract --from=$VER --to=release-image

ls release-image/*samples*

less release-image/*samples*06-operator.yaml

oc get pods -n openshift-cluster-samples-operator

oc logs pod/cluster-samples-operator-5d7d5b94b6-rw284 -n openshift-cluster-samples-operator --all-containers

oc get events -n openshift-cluster-samples-operator


```



```
lab operators-review start

cd ~/DO380/labs/operators-review

oc apply -f metering_operator.yaml

sed -i 's/^ind/kind/' metering_operator.yaml

cat metering_operator.yaml


oc apply -f metering_operator.yaml

oc get sub/lab-openshift-metering -n openshift-metering

oc delete project openshift-cluster-storage-operator

watch oc get project openshift-cluster-storage-operator



```
