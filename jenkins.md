

```
oc new-app --template jenkins-persistent

curl --user 'jenkins-user:token' https://jenkins-host/resource-path

```

```
lab gitops-deploy start

git clone https://github.com/ocplab/gitops-deploy.git

cp ~/DO380/labs/gitops-deploy/hello/* ~/gitops-deploy
cd ~/gitops-deploy
ls

oc new-app
less Jenkinsfile


git add *
git commit -m 'sample app and pipeline'
git push


oc login -u developer -p developer
oc new-project gitops-deploy

oc get template -n openshift | grep jenkins

oc process jenkins-persistent --parameters -n openshift

oc new-app --template jenkins-persistent

oc login -u admin -p redhat

oc adm policy add-cluster-role-to-user self-provisioner -z jenkins -n gitops-deploy

oc login -u developer -p developer

 oc project gitops-deploy


```


