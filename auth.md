

```

oc create secret generic ldap-secret --from-literal=bindPassword=${LDAP_ADMIN_PASSWORD} -n openshift-config

oc create configmap ca-config-map --from-file=ca.crt=<(curl http://idm.ocp-${GUID}.example.com/ipa/config/ca.crt) -n openshift-config

apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: ldapidp
  mappingMethod: claim
  type: LDAP
  ldap:
  attributes:
    id:
    - dn
    email:
    - mail
    name:
    - cn
    preferredUsername:
    - uid
  bindDN: "uid=admin,cn=users,cn=accounts,dc=ocp4,dc=example,dc=com"
  bindPassword:
    name: ldap-secret
  ca:
    name: ca-config-map
  insecure: false
  url: "ldaps://idm.ocp4.example.com/cn=users,cn=accounts,dc=ocp4,dc=example,dc=com?uid"


oc apply -f tmp/ldap-cr.yml
oc login -u admin -p ${LDAP_ADMIN_PASSWORD}
oc whoami
oc get pods -n openshift-authentication
oc logs deployment.apps/oauth-openshift

oc get user
oc get identity

oc adm policy add-cluster-role-to-user cluster-admin admin



```



```
lab auth-ldap start
oc login -u admin -p redhat https://api.ocp4.example.com:6443

oc create secret generic ldap-secret --from-literal=bindPassword='Redhat123@!' -n openshift-config

curl -O http://idm.ocp4.example.com/ipa/config/ca.crt

oc create configmap -n openshift-config ca-config-map --from-file=ca.crt

cd ~/DO380/labs/auth-ldap/

vi ldap-cr.yml

apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - htpasswd:
      fileData:
        name: htpasswd-secret
    mappingMethod: claim
    name: htpasswd_provider
    type: HTPasswd
  - name: ldapidp
    mappingMethod: claim
    type: LDAP
    ldap:
      attributes:
        id:
        - dn
        email:
        - mail
        name:
        - cn
        preferredUsername:
        - uid
      bindDN: "uid=admin,cn=users,cn=accounts,dc=ocp4,dc=example,dc=com"
      bindPassword:
        name: ldap-secret
      ca:
        name: ca-config-map
      insecure: false
      url: "ldaps://idm.ocp4.example.com/cn=users,cn=accounts,dc=ocp4,dc=example,dc=com?uid"


 oc apply -f ldap-cr.yml
 watch oc get pods -n openshift-authentication
cd
oc login -u openshift-user -p openshift-user

lab auth-ldap finish


```




