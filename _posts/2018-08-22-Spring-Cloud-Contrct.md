---

layout: post
title: 契约测试之Spring Cloud Contract 
categories: [Tech]
tags: [Micro Service , Spring]
description: 随着微服务的流行，服务的数量不断增多，服务间约束如何来保证? 契约测试应运而生。

---

随着微服务的兴起，越来越多的单体应用开始向微服务拆解。服务间通过轻量级的 Restful API 或 RPC 来进行服务间调用。此时，服务间彼此调用如何来测试？在引入契约测试之前，服务间调用有两种测试方法。

![微服务调用关系图](https://git-page.oss-cn-chengdu.aliyuncs.com/spring-cloud-contract/1.png)

以上图为例，图中每个块代表一个微服务。
假如要测试图中的 V8 微服务，则可以采用如下方式进行测试：
方式一：部署所有的微服务然后执行端到端测试。
方式二：在 unit/integration test 中 mock 其他微服务。

这两种方式的优缺点也是比较明显的。
方式一：
   优点：a. 类生产环境。 b. 测试了服务间真正的连通性。
   缺点：a. 依赖太多，测一个服务需要部署全部的服务。 b. 测试时需要封锁环境，不能同时测试。c. 耗时。d. 反馈太慢。 e. 难以调试。

方式二：
   优点：a. 快速反馈。  b. 有效的解决了方式一中依赖太多的问题，不再依赖其他微服务。
   缺点：a. mock可能和实际情况不一致，因为 mock 的返回是调用端的开发人员写的，当 mock 的服务变化时，调用端其实是无感知的。 b. 导致虽然通过测试但到生产环境仍然挂，弱化了测试的保障功能。

为了解决以上的问题，契约测试应运而生。其主要用于提供快速反馈，实现了不同服务间的依赖关系解耦，同时加快服务调用者(Consumer)对服务提供者(Provider)的感知，避免部署完成后才发现服务与服务间接口调用不一致的问题。在契约测试中，消费端不需要再自己实现 mock，而是会实际的访问服务，只是该服务是根据契约生成的 json 文件利用 WireMock 启动的一个 stub。

![](https://git-page.oss-cn-chengdu.aliyuncs.com/spring-cloud-contract/2.png)

在测试金字塔中，契约测试应该是介于端到端测试和集成测试之间的一个位置。属于将端到端测试提前的一类测试。

![测试金字塔](https://git-page.oss-cn-chengdu.aliyuncs.com/spring-cloud-contract/test-pyramid.jpg)

其典型场景主要有两种：
​	▪	内外部系统间测试
​	▪	前后端分离测试

契约测试中涉及的角色主要有两个

Consumer：使用测试替身（Test Double)，去掉直接对外部的依赖，按契约定义内容返回内容。Provider：使用契约作为测试用例，判断对应URL的返回是否符合预期。

其对应的驱动方式也有两种：PDC 和 CDC。
PDC(Provider Driven Contract) : 即服务提供者负责提供契约。这种方式很直观，但存在的问题是功能提供方的 API 返回可能无法满足所有 API 调用者的需求。由于服务提供者提供的服务主要是供消费者使用的，但提供者又无法完全合理的预估所有消费者的使用需求，为了解决这个问题，后来契约测试的驱动方式变为CDC(Consumer Driven Contract) ，即定义契约的职责被移交给了消费者，当服务者拉取最新的契约发现契约测试失败时，就会及时的实现新的契约。服务者和消费者始终基于同一份契约，也可以说是基于约束进行开发，这样就将服务间依赖测试提早，实现快速反馈但又能独立开发。

### 实际使用

#### Provider 端

1. 添加 contract-verifier 依赖。

```yaml
ext {
	springCloudVersion = 'Finchley.SR1'
}

dependencies {
	testCompile('org.springframework.cloud:spring-cloud-starter-contract-verifier')
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
	}
}
```

2. 添加契约测试插件(这里使用的插件是 2.0.0 版本，较老的版本不支持 yaml 格式的契约)

```yaml
buildscript {
	repositories {
		maven { url 'http://repo.spring.io/plugins-release/' }
	}
	dependencies {
		classpath "org.springframework.cloud:spring-cloud-contract-gradle-plugin:2.0.0.RC1"
	}
}

apply plugin: 'spring-cloud-contract'

contracts {
	baseClassForTests = "com.test.SccBaseTest"
}
```

添加该插件后将多出以下可执行任务。

![](https://git-page.oss-cn-chengdu.aliyuncs.com/spring-cloud-contract/verify-test.png)

执行 ./gradlew  build 时会自动调用这些方法，生成服务端契约测试类，以及用于 stub 的 json 文件和 jar 包等。测试类可以在 build/generated-test-sources 目录下找到。当然你也可以按需求单独执行任一条命令。

最后一个 contracts 配置指定了所有生成的契约测试都会继承的 Base 类。该类里需要对要测试的Controller 进行注册。

```java
public abstract class SccBaseTest {
	@Before
	public void setUp() throws Exception {
		RestAssuredMockMvc.standaloneSetup(UserController.class);
	}
}
```

还可以配置一系列的 Base类，此时使用配置

```yaml
packageWithBaseClasses = 'com.test.contract'
```

其指定了契约基类的目录，契约默认继承契约所在最后两级报名组成的base类。

如存在契约 user.yml, 其所在目录为 resources/contracts/authorization/user目录下，则生成的测试类默认继承 AuthorizationUserBase 类。需要定义该类并在其中完成对应 controller 的注册。

3. 添加契约

契约默认存储在 resources/contracts 目录下。

契约样例

user.yml

```yaml
request:
  method: GET
  url: /users
response:
  status: 200
  bodyFromFile: user.json
  headers:
    Content-Type: application/json;charset=UTF-8
```

user.json

```json
[
  {
    "username": "Zhang san",
    "country": "China"
  },
  {
    "username": "Steven Jobs",
    "country": "US"
  }
]
```

#### Consumer 端

1. 添加 contract-stub-runner 依赖

```yaml
ext {
	springCloudVersion = 'Finchley.SR1'
}

dependencies {
	testCompile('org.springframework.cloud:spring-cloud-starter-contract-stub-runner')
}

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
	}
}
```

2. 在测试上添加注释表明需要使用 WireMock 启动 stub。(对应 Spring Cloud Contract 2.0.x 以上的版本)

```java
@SpringBootTest
@RunWith(SpringRunner.class)
@AutoConfigureStubRunner(ids = {"com.test:SccProducer:+:stubs:8080"}, stubsMode = StubRunnerProperties.StubsMode.LOCAL)
public class TestUser {

	@Autowired
	RestTemplate restTemplate;

	@Test
	public void test_get_users() {
		final String users = restTemplate.getForObject("/users", String.class);
		final String expectedUsers = "[{\"username\":\"Zhang san\"}]";
		assertThat(users).isEqualTo(expectedUsers);
	}
}
```

其中 ids 属性中， com.test 表示 Provider 的 group id, SccProducer 表示其 artifact id, + 表示 latest 版本。将 stubs 分类器注册到 8080 端口。

stubsMode 共有三种模式: 

CLASSPATH -- 将在classpath 中获得 stubs。

 LOCAL — 将从本地仓库如 .m2 中获得 stubs。

 REMOTE — 将从远端获得 stubs。

stubsMode 的默认值为 CLASSPATH。本例中使用了 LOCAL。故还需将 stubs 发布到local maven。

### 将契约 publish 到本地仓库

在 Provider 端到 build.gradle 中添加

```yaml
apply plugin: 'maven-publish'

publishing {
	publications {
		maven(MavenPublication) {
			from components.java
			groupId = 'com.test'
			artifactId = jar.baseName
			version = '1.0'
		}
	}
}
```

添加该插件后将就会有以下可执行的任务，可以通过 publishStubsPublicationToMavenLocal 将stubs 发布到本地 maven 仓库。

![](https://git-page.oss-cn-chengdu.aliyuncs.com/spring-cloud-contract/publish.png)

