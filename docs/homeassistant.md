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

After deployment, the internal application address is `http://<instance-hostname>:8123`. Complete onboarding to create the owner account. No credentials belong in the manifest. Browsers enforce HTTPS for `.app` domains, including LAN names; a private IP address is usable only from the LAN or a connected VPN.

The HTTP health check requests `/auth/providers` and expects status 200. This upstream authentication-provider endpoint is available without authentication and was verified on the deployed 2026.10.4 instance. It checks the HTTP application path, not the operation of integrations or automations. The manifest does not request HomeStack application telemetry or heartbeat injection; container metrics are provided independently by Proxmox and the PVE agent when configured.

The manifest declares a Pangolin resource named `Home Assistant`, using the same HTTP port and health check. Its suggested public subdomain is `ha`; choose the public domain and Pangolin site in HomeStack. Pangolin provides HTTPS on the public address and forwards to port 8123 internally. Updating this repository alone does not publish or reconcile a running resource; apply the reviewed Pangolin publication through HomeStack for an existing deployment. Do not point probes at nonexistent HomeStack-specific endpoints.

Home Assistant must also trust the actual reverse proxy. For Home Assistant 2026.8 and newer, configure **Settings > System > Network > HTTP server**, enable **Trust X-Forwarded-For**, and add only the actual proxy source addresses or appropriate narrowly scoped proxy network to **Trusted proxies**. Determine these addresses from the deployed Pangolin topology; they are private deployment settings and do not belong in the manifest. Do not trust all addresses. Saving the settings restarts Home Assistant and requires confirmation within five minutes. See the [upstream HTTP documentation](https://www.home-assistant.io/integrations/http/#reverse-proxies).

Until proxy trust is configured, requests carrying `X-Forwarded-For` return HTTP 400; this was observed on the deployed instance. Publishing the Pangolin resource alone does not fix that prerequisite. For first-time remote setup, establish private access such as a VPN or SSH tunnel to configure proxy trust before relying on the public route. Keep Home Assistant's own authentication enabled as well as Pangolin's access controls; no authentication bypass paths are declared.

## Compatibility boundaries

This is Home Assistant Container, not Home Assistant OS: it does not include Supervisor or managed apps/add-ons. Deploy supporting services separately.

The manifest declares `runtime.devices.dev0` for the operator's Sonoff Zigbee
USB dongle, using its stable `/dev/serial/by-id/` path, UID/GID `0:0`, mode
`0660` and write access, as explicitly requested for this deployment. This
hardware identifier is deployment-specific; change it for another adapter.
The device must be attached to the selected Proxmox node. Use a HomeStack
version supporting schema-3 `runtime.devices` and API root-disk replacement.
The declared device must already be attached in Proxmox with matching settings.
HomeStack updates its image by switching only the root disk, keeping the same
container, `/config` volume and `dev0` mapping. No SSH or root login is used.
The deploy token cannot attach new devices or change their permissions; perform
such changes in Proxmox first. New deployments with device declarations require
that attachment capability and are refused with the current deploy token.

Bluetooth, host D-Bus mounts, and privileged operation are not declared. The upstream Docker example includes host networking and privileged mode, with an optional D-Bus mount for Bluetooth. Those Docker flags are not a HomeStack LXC configuration recipe. The operator's deployed 2026.10.4 instance was observed serving HTTP in unprivileged Proxmox LXC; this is not verification of every integration or image version. Device integrations and multicast discovery depend on the actual network and device access; declaring port 8123 does not enable them.

Initial checks covered YAML syntax and a manual review against the local HomeStack schema. Subsequent read-only checks of the operator's deployed 2026.10.4 instance confirmed HTTP responses, unprivileged LXC operation, a separate `/config` volume, and filesystem metrics. The `/auth/providers` probe returned 200; a request carrying an untrusted forwarded address returned 400. No Pangolin publication, proxy-trust changes, container restarts, image replacement, or persistent-data recovery tests were performed by these checks. Resource defaults remain unmeasured starting allocations.

## References

- [Home Assistant Container installation](https://www.home-assistant.io/installation/linux/#install-home-assistant-container)
- [Home Assistant installation types](https://www.home-assistant.io/installation/)
- [HomeStack manifest contract](https://github.com/hipotures/homestack/blob/main/.agents/skills/homestack-service/manifest-schema.md)
