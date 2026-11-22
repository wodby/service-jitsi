# Jitsi Meet service for Kubernetes on Wodby

Run Jitsi Meet video meetings on Kubernetes with Wodby.

This repository defines the Wodby service manifests and operational
configuration for Jitsi Meet.

- [Browse Wodby services](https://wodby.com/services)
- [Wodby service documentation](https://wodby.com/docs/2.0/services/)
- [Service manifest reference](https://wodby.com/docs/2.0/services/template/)

## Wodby stacks using this service

- [Jitsi Meet application stack](https://github.com/wodby/stack-jitsi)

## Service overview

| Property | Manifest configuration |
| --- | --- |
| Service name | `jitsi` |
| Type | Application service |
| Versions | `2` |
| Endpoints | `jitsi`: `http` 8000, `media` 10000/UDP |
| Settings | Moderator username, Moderator password (required), Allow guests, Media IP address |
| Workloads | main: web (primary), prosody, jicofo, jvb |
| Chart | [`wodby/stateful`](https://github.com/wodby/charts/tree/main/stateful) |
| Images | Jitsi's own `ghcr.io/jitsi/*` images |

## Use this service

Use this service through [Jitsi Meet application stack](https://github.com/wodby/stack-jitsi), or reference `jitsi`
from a custom Wodby stack. Publish the `media` port after the app is created: browsers send audio and video to it over
UDP.

A service is a reusable component and does not deploy by itself. The stack
defines its links, settings, versions, resources, and relationship to the rest
of the application.

## Maintain a custom version

1. Fork this repository.
2. Edit the service manifest and referenced files.
3. Import the repository as a [Git-backed service](https://wodby.com/docs/2.0/services/create/#create-a-git-backed-service).
4. Reference the service from a stack manifest.

Keep service, workload, container, endpoint, link, volume, config, and
derivative names stable unless dependent stacks and app-level overrides are
updated at the same time.

Validate the manifests with:

```bash
wodby service validate-manifest service.yml --org <org-id>
```

See the [service manifest reference](https://wodby.com/docs/2.0/services/template/) and the [managed services index](https://github.com/wodby/services).
