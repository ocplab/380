
```
lab gitops-resources start

cd ~/DO380/labs/gitops-resources
ls

oauth.yaml

resources:
- oauth.yaml
secretGenerator:
- name: htpasswd-secret
  namespace: openshift-config
  files:
  - htpasswd=htpasswd-secret-data
generatorOptions:
 disableNameSuffixHash: true
 
oc kustomize config

oc login -u admin -p redhat https://api.ocp4.example.com:6443

oc apply -k config
oc get pod -n openshift-authentication

oc login -u kustom -p redhat123
 
oc login -u admin -p redhat

oc extract secret/htpasswd-secret -n openshift-config --confirm --to /tmp/

htpasswd -D /tmp/htpasswd kustom

oc set data secret/htpasswd-secret -n openshift-config --from-file htpasswd=/tmp/htpasswd

oc get pod -n openshift-authentication

oc login -u kustom -p redhat123

oc extract secret/htpasswd-secret -n openshift-config --confirm --to /tmp/

oc get oauth cluster -o yaml > /tmp/oauth-config.yaml

diff /tmp/htpasswd config/htpasswd-secret-data

diff /tmp/oauth-config.yaml config/oauth.yaml

oc diff -k config

oc apply -k orig

cd ~

lab gitops-resources finish



```


