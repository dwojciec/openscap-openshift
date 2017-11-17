# openscap-openshift

In this demo we are using Host path, they are restricted by default in most SCCs since they provide direct access to the host. We need to grant access to an elevated SCC called `privileged`. Also use project `demo`.

1. Using `minishift` or `oc cluster up` login as `system:admin`:
```
oc login -u system:admin
```
  
2. Give grant access to `privileged` SCC:
```
oc adm policy add-scc-to-user privileged -z default -n demo
```

3. Create the secret (named `docker-registry`) to provide the credentials so the image inspector pod can pull and scan the image:
```
oc secrets new-dockercfg docker-registry \
    --docker-server=<registry-server-url> --docker-username=<username> \
    --docker-password=$(oc whoami -t) --docker-email=<email-address>
```

Note: I used a ServiceAccount due to the unlimited expiration time of the token. (see https://docs.openshift.com/container-platform/3.5/dev_guide/service_accounts.html#dev-managing-service-accounts)

```
$ oc project demo
$ oc create sa user-root
$ oadm policy add-cluster-role-to-user cluster-admin system:serviceaccount:demo:user-root
$ oc describe sa user-root
$ oc get secrets | grep user-root
user-root-dockercfg-1zm3z   kubernetes.io/dockercfg               1         14m
user-root-token-d7970       kubernetes.io/service-account-token   4         14m
user-root-token-qzpkc       kubernetes.io/service-account-token   4         14m

$ oc describe secret user-root-token-d7970 | grep token:
token:		eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZW1vIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InVzZXItcm9vdC10b2tlbi1kNzk3MCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJ1c2VyLXJvb3QiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmMjlmODhiYy1jYjcyLTExZTctYWZlNC01MjU0MDAwNDMzMDEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVtbzp1c2VyLXJvb3QifQ.SaCotn3I6fY5mRyWgtVGGqazOPVgc1wsQR703An68_YM4t8mIz58Klv1tmXUU_fFKboyQGzAa9Khe3lspeHE5x24WilD9uHb6mtUjJGStSVNnTEEnFkCduVwJYHnROpjQKTYL9pQxveuWhIKsWo3OIIS9EjijFQjpK0w0Nv890KFvItjB_qHFlvQrJ3Kq2yg4iGG-lSmNo7l1ph88_J49PbUIjfJ95uwhSAMDQpqWgOw2_mq-mEd5fAa5AgNPRfhVosoCI_We69yTc4hUW-RFTYXglTYwsw8NrxJCEx-Lp3iLxR_SZEy-9cUCMpJdhTOWI_9Rav4ZP8Zyd5X5U0Q3Y

$ oc delete secrets docker-registry
$ oc secrets new-dockercfg docker-registry   \
  --docker-server=docker-registry-default.apps.dwojciec.com  \
  --docker-username=user-root \
  --docker-password=eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZW1vIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InVzZXItcm9vdC10b2tlbi1kNzk3MCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJ1c2VyLXJvb3QiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJmMjlmODhiYy1jYjcyLTExZTctYWZlNC01MjU0MDAwNDMzMDEiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6ZGVtbzp1c2VyLXJvb3QifQ.SaCotn3I6fY5mRyWgtVGGqazOPVgc1wsQR703An68_YM4t8mIz58Klv1tmXUU_fFKboyQGzAa9Khe3lspeHE5x24WilD9uHb6mtUjJGStSVNnTEEnFkCduVwJYHnROpjQKTYL9pQxveuWhIKsWo3OIIS9EjijFQjpK0w0Nv890KFvItjB_qHFlvQrJ3Kq2yg4iGG-lSmNo7l1ph88_J49PbUIjfJ95uwhSAMDQpqWgOw2_mq-mEd5fAa5AgNPRfhVosoCI_We69yTc4hUW-RFTYXglTYwsw8NrxJCEx-Lp3iLxR_SZEy-9cUCMpJdhTOWI_9Rav4ZP8Zyd5X5U0Q3Y \
  --docker-email=test@test.com
```

4. Provide the url for the image to scan (eg. `IMAGE_URL=registry.access.redhat.com/rhel7:latest`)
```
oc process -f image-inspector-template.json \
    -p APPLICATION_NAME=image-inspector -p IMAGE_URL=registry.access.redhat.com/rhel7:latest \
    | oc create -f -
```

or execute the script `create-objects.sh` to create everything you need.

5. In the case you executed the script, you are going to have Jenkins provisioned in the project, so log into Jenkins and go to Manage Plugins and install the Openshift Client Plugin.

6. Now you can go to Builds -> Pipeline, and start the pipeline.

7. Open the result report at `<route url>/api/v1/content/results.html`

8. The app provided by the inspector provides WebDAV sharing to the content of the image scanned, so on Mac you can execute:
```
cadaver http://<route url>/api/v1/content/
```
