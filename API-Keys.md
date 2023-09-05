(Version >= 1.21.0)

API keys can be enabled to manage access to the Prometheus metrics endpoint.

By default, HTTP basic authentication is used to secure access to the Prometheus 
metrics endpoint. As soon as you add your first API key, the use of basic 
authentication for the endpoint will be permanently disabled.

## Adding an API key

Management of API keys is done through the API Keys page in settings 
This page will show you all of your API keys, including expired keys.

You can set the name of the key, as well as it's expiry date, or
optionally set the key to never expire. When you click generate, you
will be shown the API key in a pop up dialog, you should make sure to
make a copy of the key as it will not be shown again. The API key cannot
be used to access the web interface but you should treat API keys like
you treat your passwords as they can still be used to access potentially
sensitive data.


## Disabling keys

It is possible to temporarily disable keys. This can be done through the
settings page. This will prevent the key from being used for
authentication until it is enabled again.

## Expired keys

If a key has expired, it will not be automatically deleted. This is to
aid in debugging systems that suddenly stop working. You can delete an
expired key, or any key for that matter, by simply hitting the red
delete button from the settings page.

## Authenticating using an API key

Authentication is done by passing the API key in the `Authorization`
header. For example, here is a request made with curl to the `metrics`
endpoint.

```
curl -u":<key>" uptime.kuma/metrics
```

Note, the `:` is required before the key as basic authentication
requires a username and password separated by a `:`, however we don't
make use of the username field.

Here is an example config for Prometheus:

```
  - job_name: 'uptime'
    scrape_interval: 30s
    scheme: http
    static_configs:
      - targets: ['uptime.url']
    basic_auth: 
      password: <api key>
```

Note: we don't need to set a username field as it is not used.

Here is an example config for PrometheusOperator:

```
apiVersion: monitoring.coreos.com/v1
kind:   
metadata:
  name: uptime-kuma
  namespace: uptime-kuma
spec:
  endpoints:
    - basicAuth:
        password:
          key: password
          name: uptime-kuma-metrics-basic-auth
        username:
          key: username
          name: uptime-kuma-metrics-basic-auth
      interval: 60s
      path: /metrics
      port: http
      scrapeTimeout: 10s
  namespaceSelector:
    matchNames:
      - uptime-kuma
  selector:
    matchLabels:
      app.kubernetes.io/instance: uptime-kuma
      app.kubernetes.io/name: uptime-kuma
```

Note: we need to set a username field for ServiceMonitor object handled by Prometheus Operator, otherwise scrape will fail.
