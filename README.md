# How to deploy logstash to Cloud Foundry

```
SPACE_NAME=$(cat ~/.cf/config.json | jq -r .SpaceFields.Name)
TCP_DOMAIN=$(cf domains | grep tcp | head -1 | awk '{print $1}')
TCP_PORT=3381 # should be unique

cf create-route ${SPACE_NAME} ${TCP_DOMAIN} --port ${TCP_PORT}

cf push --no-route --no-start
cf set-env syslog-to-datadog DD_API_KEY ******

APP_GUID=$(cf app syslog-to-datadog --guid)
TCP_ROUTE_GUID=$(cf curl /v2/routes?q=port:${TCP_PORT} | jq -r .resources[0].metadata.guid)

cf curl /v2/apps/${APP_GUID} -X PUT -d "{\"ports\": [5514]}"
cf curl /v2/route_mappings -X POST -d "{\"app_guid\": \"${APP_GUID}\", \"route_guid\": \"${TCP_ROUTE_GUID}\", \"app_port\": 5514}"
cf start syslog-to-datadog
```