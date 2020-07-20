# Https (TLS/SSL) Demo

A sample of using Https in Spring Boot App (to secure app), and also how to consume REST endpoints using `RestTemplate`

## Server Side
1. Creating the Self-Signed Certificate using JSK (Java KeyStore):

        keytool -genkey -keyalg RSA -alias medium -keystore medium.jks -storepass password -validity 365 -keysize 4096 -storetype pkcs12
2. Configure the application with TLS:

        server.ssl.key-store=classpath:medium.jks
        server.ssl.key-store-type=pkcs12
        server.ssl.key-store-password=password
        server.ssl.key-password=password
        server.ssl.key-alias=medium
        server.port=8443

## Client Side
* Consuming secured REST endpoint with TLS without certificate:

        TrustStrategy acceptingTrustStrategy = (cert, authType) -> true;
        SSLContext sslContext = null;
        try {
            sslContext = SSLContexts.custom().loadTrustMaterial(null, acceptingTrustStrategy).build();
        } catch (Exception e) {
            logger.error("Catch load trust material!");
            e.printStackTrace();
        }
        SSLConnectionSocketFactory sslsf = new SSLConnectionSocketFactory(sslContext, NoopHostnameVerifier.INSTANCE);

        Registry<ConnectionSocketFactory> socketFactoryRegistry =
                RegistryBuilder.<ConnectionSocketFactory> create()
                        .register("https", sslsf)
                        .register("http", new PlainConnectionSocketFactory())
                        .build();

        BasicHttpClientConnectionManager connectionManager = new BasicHttpClientConnectionManager(socketFactoryRegistry);
        CloseableHttpClient httpClient = HttpClients.custom().setSSLSocketFactory(sslsf)
                .setConnectionManager(connectionManager).build();

        HttpComponentsClientHttpRequestFactory requestFactory = new HttpComponentsClientHttpRequestFactory(httpClient);
        ResponseEntity<String> response = new RestTemplate(requestFactory)
                .exchange(urlOverHttps, HttpMethod.GET, null, String.class);
