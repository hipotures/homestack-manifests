# HomeStack Manifests

A collection of portable service manifests for deploying third-party applications with [HomeStack](https://github.com/hipotures/homestack) on Proxmox VE.

This repository describes how applications should run: their upstream OCI images, resource defaults, ports, health checks, persistent storage, and optional exposure through Pangolin. HomeStack handles deployment and the service lifecycle.

Application source code, the HomeStack engine, and infrastructure-specific configuration belong in their respective repositories or deployment settings.

## Scope

The intended catalog includes services such as Home Assistant, Mosquitto, and Zigbee2MQTT. These are examples of the repository's purpose, not a list of currently available services.

The repository is being initialized. No deployable service manifests are included yet.

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

## Health and compatibility

Third-party images do not need to implement HomeStack-specific application endpoints. Choose a supported TCP or HTTP health check appropriate to the application. Infrastructure metrics and application-level telemetry are separate capabilities; availability depends on the deployed HomeStack components and the application.

Check the current HomeStack specification before adding a service, especially for persistence, health checks, Pangolin exposure, and hardware access. A manifest cannot add capabilities that the installed HomeStack version does not support.

## Validation

Before publishing a service, validate its manifest against the current HomeStack contract and verify application startup, health checks, and persistent data across image replacement in an isolated environment.

OCI runtime tests can verify application behavior and volume contents. Proxmox LXC deployment, volume ownership, and replacement recovery require separate verification through HomeStack on isolated Proxmox resources.
