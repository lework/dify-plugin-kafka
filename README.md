## Kafka plugin

**Author:** lework
**Version:** 0.0.1
**Type:** tool
**REPO:** [dify-plugin-kafka](https://github.com/lework/dify-plugin-kafka)

### Description

The Kafka plugin is used to send messages to Kafka topics, supports maintaining long connections to Kafka instances, and automatically reconnects when the connection is lost. Supports connecting to multiple different Kafka instances at the same time, including Kafka services that require SASL authentication.

### Features

- Send messages to the specified Kafka topic
- Support for partition routing by specifying message key
- Maintain Kafka Producer long connection
- Automatically retry when connection fails
- Support SASL/SSL secure connection
- **Support connecting to multiple different Kafka clusters**
- **Support dynamically specifying SASL authentication information**

### Configuration parameters

#### Connection credentials

- **Bootstrap Servers** (required): Kafka server address, such as `broker1:9092,broker2:9092`
- **Security Protocol** (optional): Security protocol, optional values: PLAINTEXT, SSL, SASL_PLAINTEXT, SASL_SSL
- **SASL Mechanism** (optional): SASL authentication mechanism, optional values: PLAIN, SCRAM-SHA-256, SCRAM-SHA-512
- **SASL Username** (optional): SASL username
- **SASL Password** (optional): SASL password

#### Tool parameters

- **Topic** (required): Kafka topic name to send messages

- **Message** (required): Message content to send

- **Key** (optional): Message key, used for partition routing

- **Bootstrap Servers** (optional): Specific Kafka server address, overwriting the configuration in the credentials

- **Security Protocol** (optional): Specific security protocol, overwriting the configuration in the credentials

- **SASL Mechanism** (optional): Specific SASL mechanism, overwriting the configuration in the credentials

- **SASL Username** (optional): Specific SASL username, used with SASL Password

- **SASL Password** (optional): Specific SASL password, used with SASL Username

### Secure connection configuration

#### Basic security configuration

For Kafka services that require secure connections, you can set the following parameters:

```
Security Protocol: SASL_SSL
SASL Mechanism: PLAIN (or SCRAM-SHA-256, SCRAM-SHA-512)
SASL Username: your_username
SASL Password: your_password
```

#### Important Notes

1. When the SASL username and password are configured, the plug-in automatically checks the security protocol settings. If it is not set to a protocol starting with SASL\_, it will automatically adjust to SASL_SSL.
2. If the SASL username and password are configured but the SASL mechanism is not specified, the PLAIN mechanism will be used by default.
3. When customizing the server address, the original security configuration (including SASL authentication information) will be retained by default.
4. Both the server address and SASL authentication information can be overwritten in the tool parameters.

#### Example configuration

Confluent Cloud cluster connection example:

```
Bootstrap Servers: pkc-xxxxx.region.aws.confluent.cloud:9092
Security Protocol: SASL_SSL
SASL Mechanism: PLAIN
SASL Username: your_api_key
SASL Password: your_api_secret
```

Self-hosted Kafka cluster SCRAM authentication example:

```
Bootstrap Servers: kafka1:9093,kafka2:9093
Security Protocol: SASL_SSL
SASL Mechanism: SCRAM-SHA-256
SASL Username: your_username
SASL Password: your_password
```

### Connect to multiple Kafka instances

This plugin supports connecting to multiple different Kafka instances in the Dify application. There are two ways to achieve this:

#### Method 1: Specify custom connection parameters in tool parameters

When calling the tool, you can specify specific Kafka connection information through parameters, which will override the configuration in the plugin credentials. For example:

```
Send the message "Hello, Kafka!" to the Kafka topic "test-topic" using the server "dev-kafka:9092"
```

Using this method, you can send messages to different Kafka clusters in the same process without creating a separate plugin configuration for each cluster.

##### Custom SASL authentication information

You can also specify specific SASL authentication information when calling:

```
Send the message "Hello, Secure Kafka!" to the Kafka topic "secure-topic", using the server "secure-kafka:9093", the security protocol "SASL_SSL", the SASL mechanism "PLAIN", the SASL username "admin", and the SASL password "admin-secret"
```

**Note**: If you specify both the SASL username and password, you must ensure that the security protocol settings are compatible (starting with SASL\_), otherwise the plugin will automatically adjust the security protocol to SASL_SSL.

##### Partially overwrite the configuration

You can overwrite only some of the connection information, leaving other parameters unchanged:

```
Send message "Hello, Kafka!" to Kafka topic "test-topic", using server "new-kafka:9092" // Overwrite only the server address
```

```
Send message "Hello, Kafka!" to Kafka topic "test-topic", SASL username "new-user", SASL password "new-password" // Overwrite only the authentication information
```

#### Method 2: Configure multiple plugins for different environments

Another method is to create multiple Kafka plugin configurations for different environments, each with different credentials. For example, you can create three different plugin instances:

1. Kafka-Dev: Configured to connect to the development environment (bootstrap.servers: dev-kafka:9092)

2. Kafka-Test: Configured to connect to the test environment (bootstrap.servers: test-kafka:9092)

3. Kafka-Prod: Configured to connect to the production environment, using SASL authentication (bootstrap.servers: prod-kafka:9092)

### Example Usage

In the Dify application, you can call the tool in the following format:

```
Send the message "Hello, Kafka!" to the Kafka topic "test-topic"
```

Use a custom server:

```
Send the message "Hello, Kafka!" to the Kafka topic "test-topic", using the server "custom-kafka:9092"
```

Use custom SASL authentication:

```
Send the message "Hello, Kafka!" to the Kafka topic "test-topic", using the server "custom-kafka:9092"
```

Use custom SASL authentication:

```
Send the message "Hello, Kafka!" to the Kafka topic "test-topic", using the server "custom-kafka:9092" Topic "secure-topic" sends message "Hello, Secure Kafka!", using server "secure-kafka:9093", security protocol "SASL_SSL", SASL mechanism "PLAIN", SASL username "admin", SASL password "admin-secret"
```

Complete example:

```
Send message "{"user_id": 123, "action": "login"}" to Kafka topic "user-events", using key "user123", using server "prod-kafka:9092,prod-kafka2:9092"
```

### Development and debugging

```bash
# Install dependencies
pip install -r requirements.txt

# Run debug
python -m main

# Package plugin
dify plugin package ./kafka
```
