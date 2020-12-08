
# Export / Import from Red Hat SSO

## Export

From the source project, scale down the sso application

`oc scale --replicas=0 dc/sso`

Add a JAVA_OPTS_APPEND environment variable to trigger the export 

`oc set env dc/sso -e "JAVA_OPTS_APPEND=-Dkeycloak.migration.action=export -Dkeycloak.migration.provider=singleFile -Dkeycloak.migration.file=/tmp/demorealm-export.json"`

Scale the deployment back to one pod

`oc scale --replicas=1 dc/sso`

Monitor the logs for the export to complete

`oc logs -f   $(oc get pod -l deploymentConfig=sso -o jsonpath="{.items[0].metadata.name}")`

Once the export is complete, copy the file from the pod to local machine

`oc rsync $(oc get pod -l deploymentConfig=sso -o jsonpath="{.items[0].metadata.name}"):/tmp/demorealm-export.json .`

Scale down the deployment

`oc scale --replicas=0 dc/sso`

Remove the JAVA_OPTS_APPEND env variable

`oc set env dc/sso -e "JAVA_OPTS_APPEND="`

Scale the sso deployment back to one pod

`oc scale --replicas=1 dc/sso`

## Import
Switch to destination project

Create a config map using the export file

`oc create configmap sso-import --from-file=./demorealm-export.json`

Attach the config map as a volume

`oc set volumes dc/sso --add --overwrite=true --name=sso-import --mount-path=/tmp/upload -t configmap --configmap-name=sso-import`

Scale down the deployment to 0

`oc scale --replicas=0 dc/sso`

Update the JAVA_OPTS_APPEND env variable to trigger the import

`oc set env dc/sso -e "JAVA_OPTS_APPEND=-Dkeycloak.migration.action=import -Dkeycloak.migration.provider=singleFile -Dkeycloak.migration.file=/tmp/upload/demorealm-export.json"`

Scale up the deployment to 1 replica

`oc scale --replicas=1 dc/sso`

Monitor the logs for the import to complete

`oc logs -f   $(oc get pod -l deploymentConfig=sso -o jsonpath="{.items[0].metadata.name}")`

Test the login and realm settings

Scale down the deployment

`oc scale --replicas=0 dc/sso`

Remove the JAVA_OPTS_APPEND env variable

`oc set env dc/sso -e "JAVA_OPTS_APPEND="`

Scale the sso deployment back to one pod

`oc scale --replicas=1 dc/sso`