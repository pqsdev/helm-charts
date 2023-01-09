# CHAR DE EJEMPLO PARA CLUSTER DE RABBIT MQ

Genera un nuevo RabbitMQ cluster utilizando un Helm Chart

[[_TOC_]]

## Preparacion cluster

Tiene que tener instlador el cluster operator y el topology operator **FUNCIONANDO**

- Helm en la maquina desde donde se implemente
- [GitHub - rabbitmq/messaging-topology-operator](https://github.com/rabbitmq/messaging-topology-operator)
- [GitHub - rabbitmq/cluster-operator: RabbitMQ Cluster Kubernetes Operator](https://github.com/rabbitmq/cluster-operator)

## Valores

Estos son algunos valroes de ejemplo

```yaml
# Agrupa lso valores propios del Rabbit Cluster
cluster:
  # URL de la ruta donde se expone la URL de management
  managementRoute: rabbit.localdev.me
  # Cantidad de replicas
  replicas: 1
  # sobreescritura de los contenedores del cluster
  # en este caso la intalacion de un plugin nuevo
  override:
    statefulSet:
      spec:
        template:
          spec:
            containers:
              - name: rabbitmq
                volumeMounts:
                  - mountPath: /opt/rabbitmq/community-plugins
                    name: community-plugins
                env:
                  - name: TZ
                    value: America/Buenos_Aires
            volumes:
              - name: community-plugins
                emptyDir: {}
            initContainers:
              - command:
                  - sh
                  - -c
                  - curl -L -v https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/3.9.0/rabbitmq_delayed_message_exchange-3.9.0.ez --output /community-plugins/rabbitmq_delayed_message_exchange-3.9.0.ez
                image: curlimages/curl
                imagePullPolicy: IfNotPresent
                name: copy-community-plugins
                env:
                  - name: TZ
                    value: America/Buenos_Aires
                resources:
                  limits:
                    cpu: 100m
                    memory: 500Mi
                  requests:
                    cpu: 100m
                    memory: 500Mi
                terminationMessagePolicy: FallbackToLogsOnError
                volumeMounts:
                  - mountPath: /community-plugins/
                    name: community-plugins
            securityContext: {} # necesario para openshift https://www.rabbitmq.com/kubernetes/operator/using-on-openshift.html
  # limite de recursos de cada contenedor
  resources:
    limits:
      cpu: 800m
      memory: 1Gi
    requests:
      cpu: 500m
      memory: 1Gi
  # valores de configuracion de rabbit
  # nota: aca no se debe dobreescribir el default user name y password por uqe rompe todo
  rabbitmq:
    additionalPlugins:
      - rabbitmq_delayed_message_exchange
    envConfig: |
      PLUGINS_DIR=/opt/rabbitmq/plugins:/opt/rabbitmq/community-plugins
# vhosts a definir (por cada entrada crea un vhost, usuario y permisos)
vhosts:
  - name: desa
    password: soyUnPassword
```

- Crea un nuevo cluster llamado `myrabbitmq` en el namespace `fci-lab`
- Por los valores definidos en `desa.yaml`
  - Una replica
  - Timezone Buenos Aires GMT -3
  - Plug-in `rabbitmq_delayed_message_exchange`
  - Vhost : `desa` con un usuario `desa`
  - Ruta: `rabbit.localdev.me` para la consola de administracion

## Instalacion

```bash
helm install -f myValues.yaml <<chart-name-here>> ./helm/  -n <<namespace-here>> --create-namespace
```

**Ejemplo concreto de instalacion**

```bash
helm install -f myValues.yaml myrabbitmq ./helm  -n rabbit --create-namespace
```

## Actualizacion

Deben tener el nombre con el cual fue instalado `<<chart-name-here>>`. Si no lo recuerda puede hace `helm ls -n <<namespace-here>>`.

**IMPORTANTE: ** Si se modifica el chart cambiar el numero de version en `Chart.yaml`

```
helm upgrade -f myValues.yaml `<<chart-name-here>>` ./  -n rabbit
```

## Eliminar

```bash
helm uninstall  <<chart-name-here>> -n  <<namespace-here>>
```

**EJEMPLO**

```bash
helm uninstall  myrabbitmq -n rabbit
```
