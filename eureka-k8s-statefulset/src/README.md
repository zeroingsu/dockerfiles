服务端口

``server.port=8081``

指定数据中心

``eureka.datacenter=cloud``

指定环境地址，默认为test

``eureka.environment=test``

设置是否从注册中心获取注册信息（缺省true）

这里我们是集群部署，所以设置为true.如果是单点的EurekaServer，不需要同步其它EurekaServer节点的数据，需要设为false，否则启动会报错

``eureka.client.registerWithEureka=true``

设置是否将自己作为客户端注册到注册中心（缺省true）

这里为不需要（查看@EnableEurekaServer注解的源码，会发现它间接用到了@EnableDiscoveryClient）

``eureka.client.fetchRegistry=true``

设置与Eureka Server交互的地址，查询服务和注册服务都需要依赖这个地址。默认是http://localhost:8761/eureka ；多个地址可使用 , 分隔。

``eureka.client.service-url.defaultZone=http://192.168.10.1:8081/eureka,http://192.168.10.2:8081/eureka,http://192.168.10.3:8081/eureka``

client使用IP注册服务，前台展示ip:port，非docker部署情况下推荐

```
eureka.instance.preferIpAddress=true
eureka.instance.instance-id=${spring.cloud.client.ipAddress}:${server.port}
```

client使用主机名注册服务，前台展示hostname:port,k8s部署情况下推荐

``eureka.instance.instance-id=${spring.cloud.client.hostname}:${server.port}``



logging.level.com.netflix.eureka=OFF

logging.level.com.netflix.discovery=OFF



配置优化：

1、可以防止因保护模式而不将挂掉的服务踢出掉，防止ribbon负载时，轮训到挂掉的结点时，eureka因没删除结点而去访问eureka中挂掉而未删除的服务

2、可以实现快速下线快速感知快速刷新配置解析

关闭注册中心的保护机制，Eureka 会统计15分钟之内心跳失败的比例低于85%将会触发保护机制，不剔除服务提供者，如果关闭服务注册中心将不可用的实例正确剔除

``eureka.server.enable-self-preservation=false``

eureka server刷新readCacheMap的时间，注意，client读取的是readCacheMap，这个时间决定了多久会把readWriteCacheMap的缓存更新到readCacheMap上

默认30s

``eureka.server.responseCacheUpdateInvervalMs=3000``

eureka server缓存readWriteCacheMap失效时间，这个只有在这个时间过去后缓存才会失效，失效前不会更新，过期后从registry重新读取注册服务信息，registry是一个ConcurrentHashMap。

由于启用了evict其实就用不太上改这个配置了

默认180s

``eureka.server.responseCacheAutoExpirationInSeconds=180``

启用主动失效，并且每次主动失效检测间隔为3s

``eureka.server.eviction-interval-timer-in-ms=3000``

服务过期时间配置,超过这个时间没有接收到心跳EurekaServer就会将这个实例剔除

注意，EurekaServer一定要设置eureka.server.eviction-interval-timer-in-ms否则这个配置无效，这个配置一般为服务刷新时间配置的三倍

默认90s

``eureka.instance.lease-expiration-duration-in-seconds=30``

服务刷新时间配置，每隔这个时间会主动心跳一次

默认30s

``eureka.instance.lease-renewal-interval-in-seconds=10``

eureka client刷新本地缓存时间

默认30s

``eureka.client.registryFetchIntervalSeconds=5``

eureka客户端ribbon刷新时间

默认30s

``ribbon.ServerListRefreshInterval=5000``