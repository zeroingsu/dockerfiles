## kubernetes yaml file: eureka-ss.yaml
```
---
apiVersion: v1
kind: Service
metadata:
  name: eureka
  namespace: dev
  labels:
    app: eureka
spec:
  ports:
  - port: 8761
    name: server
  clusterIP: None
  selector:
    app: eureka
---
apiVersion: policy/v1beta1
kind: PodDisruptionBudget
metadata:
  name: eureka-pdb
  namespace: dev
spec:
  selector:
    matchLabels:
      app: eureka
  maxUnavailable: 1
---
apiVersion: apps/v1beta1 # for versions before 1.8.0 use apps/v1beta1
kind: StatefulSet
metadata:
  name: eureka
  namespace: dev
spec:
  selector:
    matchLabels:
      app: eureka
  serviceName: eureka
  replicas: 3
  updateStrategy:
    type: RollingUpdate
  podManagementPolicy: Parallel
  template:
    metadata:
      labels:
        app: eureka
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - eureka
              topologyKey: "kubernetes.io/hostname"
      containers:
      - name: eureka
        imagePullPolicy: Always
        image: "zeroing/eureka:alpine-spring-2.0"
        resources:
          requests:
            memory: "1Gi"
            cpu: "0.5"
        ports:
        - containerPort: 8761
          name: server
        command:
        - sh
        - -c
        - "start-eureka \
          --servers=3 \
          --conf_file=/opt/eureka/conf/application.properties \
          --server_port=8761 \
          --eureka_datacenter=office \
          --eureka_environment=dev \
          --eureka_client_registerWithEureka=true \
          --eureka_client_fetchRegistry=true \
          --eureka_instance_preferIpAddress=false \
          --eureka_server_enable_self_preservation=true \
          --eureka_server_responseCacheUpdateInvervalMs=3000 \
          --eureka_server_responseCacheAutoExpirationInSeconds=180 \
          --eureka_server_eviction_interval_timer_in_ms=3000 \
          --eureka_instance_lease_expiration_duration_in_seconds=30 \
          --eureka_instance_lease_renewal_interval_in_seconds=10 \
          --eureka_client_registryFetchIntervalSeconds=5 \
          --ribbon_ServerListRefreshInterval=5000 \
          --heap=512M"
        readinessProbe:
          httpGet:
            path: /
            port: 8761
          initialDelaySeconds: 30
          timeoutSeconds: 10
        livenessProbe:
          httpGet:
            path: /
            port: 8761
          initialDelaySeconds: 30
          timeoutSeconds: 10
      securityContext:
        runAsUser: 0
      imagePullSecrets:
      - name: registrysecret
```


**Usage: start-eureka [OPTIONS]**

Starts a Eureka server based on the supplied options.
     --servers           The number of servers in the ensemble.default 1.

     --conf_file         The directoyr where the Eureka process will store its
                         configuration. The default is /opt/eureka/conf/application.properties.
     --server_port       The port on which the Eureka process will listen for
                         requests from other servers in the ensemble. The default is 8761.

     --eureka_datacenter   Data center configuration.default cloud.

     --eureka_environment  Environmental information configuration.default test.

     --eureka_client_registerWithEureka
                         Set whether the registration information was obtained from the registry (default true).

     --eureka_client_fetchRegistry
                         Will set self as the client registration to the registration center (default true).

     --eureka_instance_preferIpAddress
                         Client use IP registration service.default false.with native deployment recommendations to 'true',with docker(include k8s) recommendations to 'false'.

     --eureka_instance_instance-id
                         Web UI show client registration infomation format.default show '${spring.cloud.client.hostname}:${server.port}'
                         with native deployment recommendations to '${spring.cloud.client.ipAddress}:${server.port}';
                         with docker(include k8s) recommendations to '${spring.cloud.client.hostname}:${server.port}'.

     --eureka_server_enable-self-preservation
                         Registry protection mechanism.default true.

     --eureka_server_responseCacheUpdateInvervalMs
                         Eureka server refresh readCacheMap time.default 30s.

     --eureka_server_responseCacheAutoExpirationInSeconds
                         Eureka readWriteCacheMap server cache expiration time.default 180s.

     --eureka_server_eviction-interval-timer-in-ms
                         Enable active failure, failure detection and each active interval is 3s.

     --eureka_instance_lease-expiration-duration-in-seconds
                         Service expiration time configuration, and more than this time did not receive the heartbeat EurekaServer will eliminate this instance.default 90s.

     --eureka_instance_lease-renewal-interval-in-seconds
                         Service configuration, the refresh time every this time will take the initiative to a heartbeat.default 30s.

     --eureka_client_registryFetchIntervalSeconds
                         Eureka client refresh local cache time.default 30s.

     --ribbon_ServerListRefreshInterval
                         Eureka client ribbon break time.default 30s.

     --heap              The maximum amount of heap to use. The format is the
                         same as that used for the Xmx and Xms parameters to the
                         JVM. e.g. --heap=1G. The default is 1G.
