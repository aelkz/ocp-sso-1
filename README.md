# Scripted Installation

## Prerequisites
Secrets needs to be manually created as "templates". The name has to match each secret respective `as-copy-of` annotation.
```
oc process -f openshift/sso73-x509-postgresql-secrets.yaml -p 'NAME=template.sso' -p 'SUFFIX=' -l part-of=rh-sso,managed-by=template,shared=true  | oc create -f -
oc process -f openshift/sso73-x509-secrets.yaml -p 'NAME=template.sso' -p 'SUFFIX=' -l part-of=rh-sso,managed-by=template,shared=true  | oc create -f -

#if you need to remove all, or re-create/update, use the label
oc delete secret -l part-of=rh-sso,shared=true

```
## Building
```
cd .pipeline && ./npmw build -- --pr=9
```
note: replace '9' with a valid pull-request number

## Deploying
```
cd .pipeline && ./npmw deploy -- --pr=9 --env=dev
```
note: replace '9' with a valid pull-request number

## Cleanup
```
#switch to the right project/namespace on OpenShift
oc delete all -l !shared,github-repo=ocp-sso,env-id=pr-9
```

# Manual Installation
If you are just looking for quickly spin up an instance of RH-SSO

1. Switch to project
```
oc project devops-sso-sandbox
```

2. Create PostgreSQL
```
oc process -f openshift/sso73-x509-postgresql-secrets.yaml -p NAME=rh-sso -p SUFFIX=-dev -l app=rh-sso-sandbox,name=postgresql,component=database,part-of=rh-sso,managed-by=template  | oc apply -f -

oc process -f openshift/sso73-x509-postgresql.yaml -p NAME=rh-sso -p SUFFIX=-dev -l app=rh-sso-sandbox,name=postgresql,component=database,part-of=rh-sso,managed-by=template | oc apply -f -
```

3. Import Image
The template requires an ImageStream in the same namespace
```
oc import-image rh-sso:1.3-7 --from=registry.access.redhat.com/redhat-sso-7/sso73-openshift:1.3-7
```

4. Create Keycloak/RH-SSO
```
oc process -f openshift/sso73-x509-secrets.yaml -p NAME=rh-sso -p SUFFIX=-dev -l app=rh-sso-sandbox,name=keycloak,component=keycloak,part-of=rh-sso,managed-by=template  | oc apply -f -

oc process -f openshift/sso73-x509.yaml -p NAME=rh-sso -p SUFFIX=-dev -p VERSION=1.3-7 -l app=rh-sso-sandbox,name=keycloak,component=keycloak,part-of=rh-sso,managed-by=template | oc apply -f -
```

5. Delete everything
```
oc delete rc,svc,dc,route,pvc,secret -l app=rh-sso-sandbox
```

# Reference:
https://access.redhat.com/documentation/en-us/red_hat_single_sign-on/7.3/html/red_hat_single_sign-on_for_openshift/get_started#Example-Deploying-SSO
