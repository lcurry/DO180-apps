To attempt to run on openshift 
$ oc new-app --as-deployment-config=true --strategy=docker https://github.com/lcurry/DO180-apps.git#container-build  --context-dir=03-dockerfile-optimize 
Note '--as-deployment-config' so that create this application as a deployment config, which allows for hooks and custom strategies e.g. the custom serviceaccountname. 


* The parent containerfile requires root privs.  Steps needed. Create a new, specific service account, 
add it to the appropriate SCC.
$ oc create serviceaccount myserviceaccount
$ oc adm policy add-scc-to-user anyuid -z myserviceaccount
(But i found no 'dc' for my-httpd??)
Either update directly with "oc patch"
$ oc patch dc/demo-app --patch '{"spec":{"template":{"spec":{"serviceAccountName": "myserviceaccount"}}}}'
Or, Edit the DeploymentConfig object: 
Add the serviceAccount and serviceAccountName parameters to the spec field of the dc, and specify
the service account you want to use:
...
$ oc edit dc/<deployment_config>
spec:
    securityContext: {}
    serviceAccount: <service_account>

Once the deployment configuration has been updated, trigger a new deployment if necessary.
$ oc rollout latest my-httpd
deploymentconfig "my-httpd" rolled out

