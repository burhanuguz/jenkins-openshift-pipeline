
# Jenkins Pipeline for deploying Apps different Environments on Openshift
## Prerequisites
- [OpenShift Client Jenkins Plugin](https://plugins.jenkins.io/openshift-client)
## Overview
- In this explanation it is assumed that there are three different environments(dev, test, prod) in Openshift Cluster as namespaces.
- Each environment assumed to be exactly same. They are considered to have different parameters for each environment. 
- For each environment a branch is created. **master** branch is considered to be PROD, other branches are created with same names as environments. **test** branch for **TEST** and **dev** branch for **DEV** environment.
- For each environment only one template file is used and environments variables only change.
- This template file can be upgraded on DEV branch and  Because template is used in this example it would be applicable for much more complicated scenarios.  
- - -
### Explanation
- Create **Namespaces** for each environments such as; **example-dev, example-test, example-prod** and one for serviceaccount **example**.
```bash
oc create namespace example
oc create namespace example-dev
oc create namespace example-test
oc create namespace example-prod
```
- Then create **Serviceaccount** for Jenkins' OpenShift Client Plugin. 
```bash
oc create -n example serviceaccount jenkins-ca
```
- Edit permission should be given to this service account.
```bash
oc policy add-role-to-user edit system:serviceaccount:example:jenkins-sa -n example-dev
oc policy add-role-to-user edit system:serviceaccount:example:jenkins-sa -n example-test
oc policy add-role-to-user edit system:serviceaccount:example:jenkins-sa -n example-prod
```
- Open Jenkins Dashboard and then get to Manage Jenkins->Configure-System. Get to **OpenShift Client Plugin** tab and click **Add Openshift Cluster**.
- Type **Cluster Info** in that menu and then click to add credentials.

<p align="center">
  <img width="716" height="223" src="https://user-images.githubusercontent.com/59168275/92569735-a58acd80-f289-11ea-9e13-2a7ea6c8ae6b.png">
</p>

- Get the token to add it in credentials for **OpenShift Client Jenkins Plugin**.
```bash
oc get -n example secret \
$(oc get serviceaccount -n example jenkins-sa -o jsonpath='{.secrets[0].name}') \
-o jsonpath='{.data.token}' | base64 --decode
```
<p align="center">
  <img width="589" height="442" src="https://user-images.githubusercontent.com/59168275/92569714-9f94ec80-f289-11ea-886f-c7da970990de.png">
</p>

- Get **Server Certificate Authority** data from **kubeconfig** file.
  - Below down example kubeconfig file and **BASE64-encoded-certificate-authority-data** is where you get the data you need.
```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: BASE64-encoded-certificate-authority-data 
    server: https://api.<cluster_name>.<base_domain>:6443
  name: api-<cluster_name>-<base_domain>:6443
- cluster:
    certificate-authority-data: BASE64-encoded-certificate-authority-data
    server: https://api.<cluster_name>.<base_domain>:6443
  name: hb
```
```bash
echo BASE64-encoded-certificate-authority-data | base64 -d
```
- It will give certificate chain consists of **kube-apiserver-localhost-signer**, **kube-apiserver-service-network-signer**, **kube-apiserver-lb-signer**. With that TLS errors will not happen.
```bash
Example Output:
-----BEGIN CERTIFICATE-----
................................................................
................................................................
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
................................................................
................................................................
-----END CERTIFICATE-----
-----BEGIN CERTIFICATE-----
................................................................
................................................................
-----END CERTIFICATE-----
```
- Paste the certificate chain to related section and **Save** it afterwards.

![3](https://user-images.githubusercontent.com/59168275/92569726-a1f74680-f289-11ea-808f-a785ce726263.png)
- Now open Jenkins Dashboard and create new **Pipeline**. 
- Three pipeline created as **PROD**, **TEST** and **DEV** to make deploy on all environments at once. Configure it to use **Pipeline script from SCM** and also choose **git** as SCM. For each pipeline branch as in the picture. 

![4](https://user-images.githubusercontent.com/59168275/92569727-a28fdd00-f289-11ea-92a4-2bc2aa785b05.png)
- Start build for pipelines and pods will be deployed in **example-prod**, **example-test** and **example-dev** projects. 

<p align="center">
  <img width="664" height="154" src="https://user-images.githubusercontent.com/59168275/92569728-a3c10a00-f289-11ea-9b7b-d48f73719ab5.png">
</p>

![6](https://user-images.githubusercontent.com/59168275/92569733-a459a080-f289-11ea-83cc-b11f73772084.png)
- - -
### Resources
1. [https://github.com/openshift/jenkins-client-plugin](https://github.com/openshift/jenkins-client-plugin)
