## Kafka 插件

**作者:** lework
**版本:** 0.0.1
**类型:** tool
**REPO:** [dify-plugin-kafka](https://github.com/lework/dify-plugin-kafka)

### 描述

Kafka 插件用于向 Kafka 主题发送消息，支持维护 Kafka 实例的长连接，并在连接丢失时自动重连。支持同时连接多个不同的 Kafka 实例，包括需要 SASL 认证的 Kafka 服务。

### 功能特性

- 发送消息到指定的 Kafka 主题
- 支持指定消息 key 进行分区路由
- 维护 Kafka Producer 长连接
- 连接失败时自动重试
- 支持 SASL/SSL 安全连接
- **支持连接多个不同的 Kafka 集群**
- **支持动态指定 SASL 认证信息**

### 配置参数

#### 连接凭证

- **Bootstrap Servers** (必填): Kafka 服务器地址，如 `broker1:9092,broker2:9092`
- **Security Protocol** (可选): 安全协议，可选值：PLAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL
- **SASL Mechanism** (可选): SASL 认证机制，可选值：PLAIN, SCRAM-SHA-256, SCRAM-SHA-512
- **SASL Username** (可选): SASL 用户名
- **SASL Password** (可选): SASL 密码

#### 工具参数

- **Topic** (必填): 要发送消息的 Kafka 主题名称
- **Message** (必填): 要发送的消息内容
- **Key** (可选): 消息的 key，用于分区路由
- **Bootstrap Servers** (可选): 特定的 Kafka 服务器地址，覆盖凭证中的配置
- **Security Protocol** (可选): 特定的安全协议，覆盖凭证中的配置
- **SASL Mechanism** (可选): 特定的 SASL 机制，覆盖凭证中的配置
- **SASL Username** (可选): 特定的 SASL 用户名，与 SASL Password 一起使用
- **SASL Password** (可选): 特定的 SASL 密码，与 SASL Username 一起使用

### 安全连接配置

#### 基本安全配置

对于需要安全连接的 Kafka 服务，可以设置以下参数：

```
Security Protocol: SASL_SSL
SASL Mechanism: PLAIN (或者 SCRAM-SHA-256, SCRAM-SHA-512)
SASL Username: your_username
SASL Password: your_password
```

#### 重要说明

1. 当配置了 SASL 用户名和密码时，插件会自动检查安全协议设置，如果未设置为 SASL\_开头的协议，会自动调整为 SASL_SSL。
2. 如果配置了 SASL 用户名和密码但未指定 SASL 机制，默认会使用 PLAIN 机制。
3. 自定义服务器地址时，默认会保留原有的安全配置（包括 SASL 认证信息）。
4. 可以在工具参数中同时覆盖服务器地址和 SASL 认证信息。

#### 示例配置

Confluent Cloud 集群连接示例：

```
Bootstrap Servers: pkc-xxxxx.region.aws.confluent.cloud:9092
Security Protocol: SASL_SSL
SASL Mechanism: PLAIN
SASL Username: your_api_key
SASL Password: your_api_secret
```

自托管 Kafka 集群 SCRAM 认证示例：

```
Bootstrap Servers: kafka1:9093,kafka2:9093
Security Protocol: SASL_SSL
SASL Mechanism: SCRAM-SHA-256
SASL Username: your_username
SASL Password: your_password
```

### 连接多个 Kafka 实例

本插件支持在 Dify 应用中连接多个不同的 Kafka 实例。有两种方式可以实现：

#### 方式一：在工具参数中指定自定义连接参数

在调用工具时，可以通过参数指定特定的 Kafka 连接信息，这将覆盖插件凭证中的配置。例如：

```
向 Kafka 主题 "test-topic" 发送消息 "Hello, Kafka!"，使用服务器 "dev-kafka:9092"
```

使用这种方式，您可以在同一个流程中向不同的 Kafka 集群发送消息，而无需为每个集群创建单独的插件配置。

##### 自定义 SASL 认证信息

您也可以在调用时指定特定的 SASL 认证信息：

```
向 Kafka 主题 "secure-topic" 发送消息 "Hello, Secure Kafka!"，使用服务器 "secure-kafka:9093"，安全协议 "SASL_SSL"，SASL机制 "PLAIN"，SASL用户名 "admin"，SASL密码 "admin-secret"
```

**注意**：如果同时指定了 SASL 用户名和密码，必须确保安全协议设置兼容（以 SASL\_ 开头），否则插件会自动将安全协议调整为 SASL_SSL。

##### 部分覆盖配置

您可以只覆盖部分连接信息，保留其他参数不变：

```
向 Kafka 主题 "test-topic" 发送消息 "Hello, Kafka!"，使用服务器 "new-kafka:9092"  // 只覆盖服务器地址
```

```
向 Kafka 主题 "test-topic" 发送消息 "Hello, Kafka!"，SASL用户名 "new-user"，SASL密码 "new-password"  // 只覆盖认证信息
```

#### 方式二：为不同环境配置多个插件

另一种方式是为不同的环境创建多个 Kafka 插件配置，每个配置使用不同的凭证。例如，您可以创建三个不同的插件实例：

1. Kafka-Dev：配置连接到开发环境 (bootstrap.servers: dev-kafka:9092)
2. Kafka-Test：配置连接到测试环境 (bootstrap.servers: test-kafka:9092)
3. Kafka-Prod：配置连接到生产环境，使用 SASL 认证 (bootstrap.servers: prod-kafka:9092)

### 示例用法

在 Dify 应用中，可以通过以下格式调用该工具：

```
向 Kafka 主题 "test-topic" 发送消息 "Hello, Kafka!"
```

使用自定义服务器：

```
向 Kafka 主题 "test-topic" 发送消息 "Hello, Kafka!"，使用服务器 "custom-kafka:9092"
```

使用自定义 SASL 认证：

```
向 Kafka 主题 "secure-topic" 发送消息 "Hello, Secure Kafka!"，使用服务器 "secure-kafka:9093"，安全协议 "SASL_SSL"，SASL机制 "PLAIN"，SASL用户名 "admin"，SASL密码 "admin-secret"
```

完整示例：

```
向 Kafka 主题 "user-events" 发送消息 "{"user_id": 123, "action": "login"}"，使用 key "user123"，使用服务器 "prod-kafka:9092,prod-kafka2:9092"
```

### 开发和调试

```bash
# 安装依赖
pip install -r requirements.txt

# 运行调试
python -m main

# 打包插件
dify plugin package ./dify-plugin-kafka
```

## Cursor 提示词

```
你是一个Dify插件开发专家，我已经初始化好插件项目了。现在你要实现以下需求：
## 插件目的
往 kafka topic 中写入数据

## 插件需求
1. 支持连接不同的kafka实例
2. 全局维护kafka不同实例的长链接，在连接丢失时自动重连
3. 在工具参数中增加了可选的kafka连接参数，允许在调用工具时指定特定的Kafka服务器地址
4. kafka 连接支持身份验证的SASL机制

## 要求
先整理下我的需求，进行分析评审，先找出可行的方案在进行编写代码。
python kafka sdk你可以参考 https://docs.confluent.io/kafka-clients/python/current/overview.html
```
