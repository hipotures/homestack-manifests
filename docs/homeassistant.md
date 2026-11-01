# Home Assistant

This service uses the official `ghcr.io/home-assistant/home-assistant` image unchanged, with its upstream entrypoint and command. It targets HomeStack schema 3 and a single unprivileged Proxmox OCI application container. Docker inside LXC is not required.

## Deployment settings

- Use a HomeStack version supporting schema 3 and configurable persistent mount paths.
- Select a published numeric upstream release, such as `YEAR.MONTH.PATCH`, using HomeStack's image selection. The manifest specifies only the repository, not the moving `stable` or `latest` tag.
- Choose the node, storage, network, hostname, and domain in HomeStack. Provide working DNS and outbound access for the integrations you use.
- Set the `Time zone` application input to an IANA name such as `Europe/Warsaw`; the default is `Etc/UTC`. The input supplies `TZ` to the image. Set the application's location and time zone during onboarding too. The manifest checks the name's syntax, not membership in the IANA database.

The initial allocations are 2 vCPUs, 2 GiB RAM, no swap, an 8 GiB disposable root, and a 10 GiB persistent volume. These are editable starting allocations, not measured requirements or a capacity guarantee. Adjust them for the selected image, integrations, database history, and backups.

## Data and updates

The persistent volume is mounted directly at `/config`, where Home Assistant keeps configuration and its default local database. The rest of the root filesystem is disposable. Keep any additional durable application files inside `/config`, or in an explicitly configured external service.

Use one instance: additional instances have independent configuration and data; they are not replicas. Back up application data before an image update. HomeStack image replacement is intended to retain the original persistent volume; this manifest has not undergone runtime or Proxmox replacement verification. Application upgrades may change the database, so reverting only the image is not a complete rollback.

## Access and health

After deployment, open `http://<instance-hostname>:8123` on the LAN and complete onboarding to create the owner account. No credentials belong in the manifest.

The TCP health check only establishes that port 8123 accepts connections. It does not prove that integrations or automations work. The manifest does not request HomeStack application telemetry or heartbeat injection; container metrics are provided independently by Proxmox and the PVE agent when configured.

Pangolin exposure is disabled. Enabling it requires a verified upstream HTTP health endpoint and its expected status code, plus the appropriate Home Assistant reverse-proxy configuration. Do not point probes at nonexistent HomeStack-specific endpoints.

## Compatibility boundaries

This is Home Assistant Container, not Home Assistant OS: it does not include Supervisor or managed apps/add-ons. Deploy supporting services separately.

USB passthrough, Bluetooth, host D-Bus mounts, and privileged operation are not declared. The upstream Docker example includes host networking and privileged mode, with an optional D-Bus mount for Bluetooth. Those Docker flags are not a HomeStack LXC configuration recipe. Basic startup in unprivileged Proxmox LXC remains unverified. Device integrations and multicast discovery depend on the actual network and device access; declaring port 8123 does not enable them.

Only YAML syntax and a manual review against the local HomeStack schema were checked. No application startup, resource measurements, HTTP probes, image replacement, or live Proxmox tests were performed.

## References

- [Home Assistant Container installation](https://www.home-assistant.io/installation/linux/#install-home-assistant-container)
- [Home Assistant installation types](https://www.home-assistant.io/installation/)
- [HomeStack manifest contract](https://github.com/hipotures/homestack/blob/main/.agents/skills/homestack-service/manifest-schema.md)
