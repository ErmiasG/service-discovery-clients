# Java client for Consul service discovery

This is a library to get services information from Consul.
There are two available resolvers.

1. Using Consul HTTP API
2. Using DNS

## Table of contents
1. [Installation](#installation)
2. [Usage](#usage)

## Installation
To use the library include the dependency in your `pom.xml` file

    <dependency>
        <groupId>com.logicalclocks</groupId>
        <artifactId>service-discovery-client</artifactId>
        <version>VERSION</version>
    </dependency>

You should also add our Maven repository.

## Usage
You can query for a service definition either with Consul HTTP API or
using the DNS interface and `SRV` records. The HTTP API is more expressive
as it can filter services also with a set of tags.

Both methods share a common interface and you can create them using the
provided `Builder`.

### HTTP API

In the following example we create an HTTP client with some options
and we lookup for a service with name `my-service-name` and some tags
associated with it. If there is no service with that name and with these
tags a `ServiceNotFoundException` will be thrown.

If you have Consul HTTP API configured with TLS, you should supply a
properly configured `SSLContext`

```java
ServiceDiscoveryClient client = null;
    try {
      client = new Builder(Type.HTTP)
          .withHttpHost("localhost")
          .withHttpPort(8501)
          .withHttps()
          .withHostnameVerifier(allowAllHostnameVerifier)
          .withSSLContext(sslContextWithConfiguredKeystores)
          .build();
      
      Set<String> tags = new HashSet<>(Arrays.asList("tag0", "tag1"));
      List<Service> services = client.getService(
          ServiceQuery.of("my-service-name", tags));
    } catch (ServiceDiscoveryException ex) {
      // Handle exception
    } finally {
      if (client != null) {
        client.close();
      }
    }
```

### DNS

When HTTP is not an option or you don't have access to the required keystores
you can use DNS to query Consul, provided that it's configured correctly.

The API is the same. One **important** difference is that the service name
should be the FQDN of the service. Also, you can't query with tags.

```java
ServiceDiscoveryClient client = null;
    try {
      client = new Builder(Type.DNS)
          .withDnsHost("127.0.0.1")
          .withDnsPort(53)
          .build();
      
      List<Service> services = client.getService(
          ServiceQuery.of("my-service-name.service.domain", Collections.emptySet()));
    } catch (ServiceDiscoveryException ex) {
      // Handle exception
    } finally {
      if (client != null) {
        client.close();
      }
    }
```