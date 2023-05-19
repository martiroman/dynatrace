# Proyecto KEPTN
## Instalar el servicio de Dynatrace

```
helm upgrade --install dynatrace-service -n keptn https://github.com/keptn-contrib/dynatrace-service/releases/download/0.27.0/dynatrace-service-0.27.0.tgz --set dynatraceService.config.keptnApiUrl=$KEPTN_API_URL --set dynatraceService.config.keptnBridgeUrl=$KEPTN_BRIDGE_URL
```

Crear secret con las credenciales de Dynatrace

```
DT_TENANT=yourtenant.live.dynatrace.com
DT_API_TOKEN=yourAPItoken
keptn create secret dynatrace --scope=dynatrace-service --from-literal="DT_TENANT=$DT_TENANT" --from-literal="DT_API_TOKEN=$DT_API_TOKEN"
```

## Crear proyecto "dynatrace" en Keptn

shipyard-quality-gates.yaml 
```
--- 
apiVersion: "spec.keptn.sh/0.2.2" 
kind: "Shipyard" 
metadata: 
  name: "dynatrace" 
spec: 
  stages: 
    - name: "quality-gate" 
      test_strategy: "performance" 
```

```
keptn create project dynatrace --shipyard=./shipyard-quality-gates.yaml --git-user=$GIT_USER --git-token=$GIT_TOKEN --git-remote-url=https://github.com/powercloudtest/dynatrace.git 
```


## Configurar el proyecto para conectar con el tenant


dynatrace.conf.yaml 

```
--- 
spec_version: '0.1.0' 
dtCreds: dynatrace 
# Opcional: Se puede crear un dashboard con las metricas y SLO
dashboard: <dashboard ID>
```

Agregar el recurso

```
keptn add-resource --project=dynatrace --resource=dynatrace.conf.yaml --resourceUri=dynatrace/dynatrace.conf.yaml 
```

Configurar el plugin

```
keptn configure monitoring dynatrace --project=dynatrace
```

* Por defecto, el servicio de Dynatrace chequea cada 60 segundos la existencia de nuevos servicios.
* Los servicios deberan tener los tags: keptn_managed y keptn_service:<service_name>


## Configurar el servicio

Una vez que Keptn levantó el servicio, se puede configurar metricas y slo en particular a través de un dashboard

dynatrace.conf.yaml 

```
--- 
spec_version: '0.1.0' 
dtCreds: dynatrace 
dashboard: <dashboard ID>
```

Agregar el recurso

```
keptn add-resource --project=dynatrace --stage=quality-gate --service=<your_service_name> --resource=dynatrace.conf.yaml --resourceUri=dynatrace/dynatrace.conf.yaml 
```


## Ejecutar una evaluación en forma manual

```
keptn trigger evaluation --project=dynatrace --stage=quality-gate --service=carts --timeframe=10m --labels=releasesVersion=0.11.1,executedBy=manual 
```
