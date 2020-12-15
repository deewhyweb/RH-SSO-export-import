
# Export / Import from Red Hat SSO

## Export

Login to the RH SSO console as administrator

Select the realm required for export.

Select the "Export" link on the left hand menu.

Ensure both "Export groups and roles" and "Export clients" are set to "On"

Click on export

The realm data will be exported to a file calles realm-export.json


## Import
Switch to destination project

Create a config map using the export file

`oc create configmap sso-import --from-file=./realm-export.json`

Attach the config map as a volume

`oc set volumes dc/sso --add --overwrite=true --name=sso-import --mount-path=/tmp/upload -t configmap --configmap-name=sso-import`

Scale down the deployment to 0

`oc scale --replicas=0 dc/sso`

Update the JAVA_OPTS_APPEND env variable to trigger the import by adding the following string.

`JAVA_OPTS_APPEND=-Dkeycloak.migration.action=import -Dkeycloak.migration.provider=singleFile -Dkeycloak.migration.file=/tmp/upload/realm-export.json"`

Scale up the deployment to 1 replica

`oc scale --replicas=1 dc/sso`

Monitor the logs for the import to complete

`oc logs -f   $(oc get pod -l deploymentConfig=sso -o jsonpath="{.items[0].metadata.name}")`

Test the login and realm settings


Remove the JAVA_OPTS_APPEND env variable

`oc set env dc/sso -e "JAVA_OPTS_APPEND="`
