

```
lab certificates-enterprise-ca start
cd ~/DO380/labs/certificates-enterprise-ca/

openssl x509 -in wildcard-api.pem -noout -subject -issuer -ext 'subjectAltName'

cat wildcard-api.pem /etc/pki/tls/certs/classroom-ca.pem > combined-cert.pem

 oc login -u admin -p redhat
 
 oc create secret tls custom-tls --cert combined-cert.pem --key wildcard-api-key.pem -n openshift-config
 
 vim apiserver-cluster.yaml
 
apiVersion: config.openshift.io/v1
kind: APIServer
metadata:
  name: cluster
spec:
  servingCerts:
    namedCertificates:
    - names:
      - api.ocp4.example.com
      servingCertificate:
        name: custom-tls

oc apply -f apiserver-cluster.yaml

 watch oc get co/kube-apiserver
 oc create configmap combined-certs --from-file ca-bundle.crt=combined-cert.pem -n openshift-config
 
  vim proxy-cluster.yaml
  
apiVersion: config.openshift.io/v1
kind: Proxy
metadata:
  name: cluster
spec:
  trustedCA:
    name: combined-certs


 oc apply -f proxy-cluster.yaml
 
 oc create secret tls custom-tls-bundle --cert combined-cert.pem --key wildcard-api-key.pem -n openshift-ingress
 
  vim ingresscontrollers.yaml

apiVersion: operator.openshift.io/v1
kind: IngressController
metadata:
  finalizers:
  - ingresscontroller.operator.openshift.io/finalizer-ingresscontroller
  name: default
  namespace: openshift-ingress-operator
spec:
  defaultCertificate:
    name: custom-tls-bundle
  replicas: 2


 oc apply -f ingresscontrollers.yaml

cd

watch oc get pods -n openshift-ingress

oc whoami --show-console

 watch oc get co/kube-apiserver
 
 oc logout
 rm -rf ~/.kube/
 
 oc login -u admin -p redhat https://api.ocp4.example.com:6443

 lab certificates-enterprise-ca finish

```


```
 oc get proxy/cluster -o jsonpath='{.spec.trustedCA.name}{"\n"}' <CONFIGMAP-NAME>
 
 oc extract configmap <CONFIGMAP-NAME> -n openshift-config --confirm
 
 less ca-bundle.crt
 
 oc set data configmap <CONFIGMAP-NAME> --from-file ca-bundle.crt=<PATH-TO-NEW-CERTIFICATE> -n openshift-config
 
 oc create configmap <CONFIGMAP-NAME>
 
  oc label configmap <CONFIGMAP-NAME> config.openshift.io/inject-trusted-cabundle=true
  
  
   oc set volume dc/<DC-NAME> -t configmap --name trusted-ca --add --read-only=true --mount-path /etc/pki/ca-trust/extracted/pem --configmap-name <CONFIGMAP-NAME>
   
   oc rsh hello-3-65qs7
   curl https://hello.apps.ocp4.example.com

 

```


```
 lab certificates-app-trust start
 
 oc login -u admin -p redhat https://api.ocp4.example.com:6443
 
 oc get proxy/cluster -o yaml
 
 oc get configmap combined-certs -n openshift-config -o jsonpath='{.data.*}' | grep Classroom
 
 oc new-project certificates-app-trust
  oc new-app --name hello1 --docker-image quay.io/redhattraining/hello-world-nginx:v1.0
  
  
   oc create route edge --service hello1 --hostname hello1-trust.apps.ocp4.example.com
   
   
   oc new-app --name hello2 --docker-image quay.io/redhattraining/hello-world-nginx:v1.0
   
   oc create route edge --service hello2 --hostname hello2-trust.apps.ocp4.example.com
   
 oc get route
 
 curl https://hello1-trust.apps.ocp4.example.com
 
 curl https://hello2-trust.apps.ocp4.example.com
oc get pods -l deployment=hello1


 oc rsh hello1-766cfb954c-hghz8
 
  curl https://hello2-trust.apps.ocp4.example.com
  
  exit
  
 curl https://hello2-trust.apps.ocp4.example.com
 
 oc label configmap ca-certs config.openshift.io/inject-trusted-cabundle=true
 
 oc get configmap ca-certs -o yaml | head -n6
 
 oc set volume deployment/hello1 -t configmap --name trusted-ca --add --read-only=true --mount-path /etc/pki/ca-trust/extracted/pem --configmap-name ca-certs


oc edit deployment/hello1

...output omitted...
  volumes:
  - configMap:
    defaultMode: 420
    items:
    - key: ca-bundle.crt
      path: tls-ca-bundle.pem
    name: ca-certs
  name: trusted-ca
  
  
  watch oc get pods -l deployment=hello1


```


```

watch oc get co/kube-apiserver


oc get apiserver/cluster -o yaml

...output omitted...
spec:
  servingCerts:
  namedCertificates:
  - names:
    - <API-URL>
    servingCertificate:
      name: <SECRET-NAME>

oc extract secret/<SECRET-NAME> -n openshift-config --confirm
openssl x509 -in tls.crt -noout -dates

oc set data secret <SECRET-NAME> --from-file tls.crt=<PATH-TO-NEW-CERTIFICATE> --from-file tls.key=<PATH-TO-KEY> -n openshift-config

oc get ingresscontroller/default -n openshift-ingress-operator -o jsonpath='{.spec.defaultCertificate.name}{"\n"}'

oc extract secret/<SECRET-NAME> -n openshift-ingress --confirm
 openssl x509 -in tls.crt -noout -dates
 
 oc set data secret <SECRET-NAME> --from-file tls.crt=<PATH-TO-NEW-CERTIFICATE> --from-file tls.key=<PATH-TO-KEY> -n openshift-config
 
  curl -k -v https://oauth-openshift.apps.ocp4.example.com 2>&1 | grep -w date
  
  oc get proxy/cluster -o jsonpath='{.spec.trustedCA.name}{"\n"}'
  
   oc set data configmap <CONFIGMAP-NAME> --from-file ca-bundle.crt=<PATH-TO-NEW-CERTIFICATE> -n openshift-config

openssl x509 -in <PATH-TO-CERTIFICATE> -noout -dates


oc get secret <SECRET-NAME> -n openshift-config -o jsonpath='{.data.*}' | base64 -di | openssl x509 -noout -serial

openssl x509 -in <PATH-TO-CERTIFICATE> -noout -serial

oc get events --sort-by='.lastTimestamp' -n openshift-kube-apiserver

oc get pods -n openshift-ingress


```

