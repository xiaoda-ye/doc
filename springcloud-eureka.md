# springcloud-eureka

其中server.port配置eureka服务器端口号。Eureka的配置属性都在开源项目spring-cloud-netflix-master中定义（spring boot连文档都没有，只能看源码了），在这个项目中有两个类[EurekaInstanceConfigBean](http://github.com/%7Bgithub-repo%7D/tree/%7Bgithub-tag%7D/spring-cloud-netflix-core/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java) 和[EurekaClientConfigBean](http://github.com/%7Bgithub-repo%7D/tree/%7Bgithub-tag%7D/spring-cloud-netflix-core/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java)，分别含有eureka.instance和eureka.client相关属性的解释和定义。从中可以看到，registerWithEureka表示是否注册自身到eureka服务器，因为当前这个应用就是eureka服务器，没必要注册自身，所以这里是false。fetchRegistry表示是否从eureka服务器获取注册信息，同上，这里不需要。defaultZone就比较重要了，是设置eureka服务器所在的地址，查询服务和注册服务都需要依赖这个地址。

```yaml
server:
  port: 8888
eureka:
  instance:
    hostname: localhost #eureka服务端的实例名称
    instance-id: springcloud-eurekaServer #自定义服务名称信息
    prefer-ip-address: true #访问路径可以显示IP地址
  client:
    register-with-eureka: false #false表示不向注册中心注册自己
    fetch-registry: false #false表示自己就是注册中心，我的职责就是维护服务实例，并不需要区检索服务
    service-url:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ #设置与eureka Server 交互的地址查询和服务注册都需要依赖这个地址。
```

