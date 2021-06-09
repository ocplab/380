

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

```
lab auth-ldapsync start
oc login -u admin -p redhat

cd ~/DO380/labs/auth-ldapsync/

wget http://idm.ocp4.example.com/ipa/config/ca.crt
vi ldap-sync.yml

kind: LDAPSyncConfig
apiVersion: v1
url: ldaps://idm.ocp4.example.com
bindDN: uid=admin,cn=users,cn=accounts,dc=ocp4,dc=example,dc=com
bindPassword: Redhat123@!
insecure: false
ca: ca.crt
rfc2307:

oc adm groups sync --sync-config ldap-sync.yml
oc new-project auth-ldapsync
 oc create -f rbac.yml
 
 vi cronjob.yml
 
 apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: group-sync
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: hello
            image: registry.redhat.io/openshift4/ose-cli:v4.5
            command:
            - /bin/sh
            - -c
            - oc adm groups sync --sync-config /etc/config/cron-ldap-sync.yml --confirm
            volumeMounts:
              - mountPath: "/etc/config"
                name: "ldap-sync-volume"
              - mountPath: "/etc/secrets"
                name: "ldap-bind-password"
          volumes:
            - name: "ldap-sync-volume"
              configMap:
                name: ldap-config
            - name: "ldap-bind-password"
              secret:
                secretName: ldap-secret
          serviceAccountName: ldap-group-syncer
          serviceAccount: ldap-group-syncer



```


