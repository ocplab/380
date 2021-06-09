

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
