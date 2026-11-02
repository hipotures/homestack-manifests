# HomeStack Manifests

A collection of portable service manifests for deploying third-party applications with [HomeStack](https://github.com/hipotures/homestack) on Proxmox VE.

This repository describes how applications should run: their upstream OCI images, resource defaults, ports, health checks, persistent storage, and optional exposure through Pangolin. HomeStack handles deployment and the service lifecycle.

Application source code, the HomeStack engine, and infrastructure-specific configuration belong in their respective repositories or deployment settings.

## Scope

The catalog currently includes a [Home Assistant manifest](docs/homeassistant.md) using the official upstream image and a persistent `/config` volume. Runtime compatibility has not yet been verified. Mosquitto and Zigbee2MQTT are examples of possible future additions.

## Repository layout

New services should follow HomeStack's schema 3 layout:

```text
.homestack/
  services/
    <service-id>/
      service.yaml
      defaults.yaml
      inputs.yaml
```

Use the [manifest specification](https://github.com/hipotures/homestack/blob/main/.agents/skills/homestack-service/manifest-schema.md) in the main HomeStack repository as the source of truth for file contents, supported fields, and validation rules.

## Manifest principles

- Prefer upstream OCI images and preserve their intended startup behavior.
- Keep manifests portable. Supply private IP addresses, container IDs, storage names, secrets, and device identifiers through deployment configuration rather than committing them here.
- Declare resource requirements, application ports, and suitable health checks explicitly.
- Keep application state on a dedicated persistent volume, separate from the disposable root filesystem.
- Document service-specific requirements and compatibility with the HomeStack version used for deployment.

## Persistent storage

Use the application's native data directory where supported by HomeStack. For example, an application that stores its state in `/config` can declare:

```yaml
persistence:
  mount: /config
  size: 10G
```

The default mount path is `/home/app` when `mount` is omitted from a configured persistence section. An empty persistence section means that no persistent volume is requested.

Image replacement must preserve the existing persistent volume and its mount path. Changing a manifest's mount path is not a data migration; existing deployments require an explicit migration plan.

## Publishing an existing service through Pangolin

A service that already runs on its own container, virtual machine or host can be published through Pangolin without HomeStack creating anything. Leave `resources` empty in `defaults.yaml`: HomeStack then creates no container and only adds a Pangolin HTTP resource that routes to the existing service. The service itself is never changed.

HomeStack routes to `<hostname>.<domain>` on the declared port through the Pangolin Site connector on the chosen node. The name must resolve through the DNS that connector uses, and the service must be reachable from it.

`service.yaml`:

```yaml
schema: 3
id: myapp
name: My App
image:
  # Required by the schema and checked against its registry, but never pulled.
  repository: docker.io/example/myapp
runtime:
  ports:
    - name: http
      port: 8080
      protocol: tcp
  # Pangolin monitors the service with this HTTP check.
  healthcheck:
    type: http
    port: http
    path: /
    expected_status: 200
persistence:
exposure:
  pangolin:
    port: http
    name: My App
    subdomain: myapp
```

`defaults.yaml`:

```yaml
schema: 3
service: myapp
# Empty: no container is created; the service is only published.
resources:
deployment:
  hostname: myapp
  instances: 1
  placement: single
  state_model: stateless
```

`inputs.yaml`:

```yaml
schema: 3
service: myapp
inputs: []
```

In the HomeStack TUI deployment of such a service:

1. **Nodes**: choose the node the service runs on; its Pangolin Site connector routes the traffic.
2. **Hostname** and **Domain**: together they form the service's local address, for example `myapp` and `home.example` for `myapp.home.example`.
3. **Pangolin**: press Enter to attach, then choose the **Pangolin domain**. The service is published as `<subdomain>.<Pangolin domain>`, for example `https://myapp.example.com`.
4. Review: it lists only the Pangolin resource, its monitoring and the public DNS record. Apply creates nothing else.

Limitations:

- Only HTTP services are supported; the resource and its health check use HTTP.
- The target must be a host name, not an IP address.
- A running HomeStack Pangolin Site connector must exist on the chosen node.

## Health and compatibility

Third-party images do not need to implement HomeStack-specific application endpoints. Choose a supported TCP or HTTP health check appropriate to the application. Infrastructure metrics and application-level telemetry are separate capabilities; availability depends on the deployed HomeStack components and the application.

Check the current HomeStack specification before adding a service, especially for persistence, health checks, Pangolin exposure, and hardware access. A manifest cannot add capabilities that the installed HomeStack version does not support.

## Validation

Before publishing a service, validate its manifest against the current HomeStack contract and verify application startup, health checks, and persistent data across image replacement in an isolated environment.

OCI runtime tests can verify application behavior and volume contents. Proxmox LXC deployment, volume ownership, and replacement recovery require separate verification through HomeStack on isolated Proxmox resources.
