# LoStack Depot

This repo serves as the backend for the LoStack Depot. It is designed to be compatible with GetHomePage and should work with DockGE.

Pull Requests are Welcome!

---

## Package Standard

LoStack implements its own labeling standard to replace Traefik's routing. There may be certain instances where it is appropriate to use Traefik labeling, but the default LoStack routing labels should be used wherever possible.

LoStack borrows GetHomePage's labeling system (and icon system), which means packages installed with LoStack will appear on GetHomePage with the correct labels and description. LoStack's built-in Dashboard is lacking in comparison to GetHomePage. A first-party plugin to generate GetHomePage dashboards per-user is in the works.

LoStack reads the labels from running containers and automatically populates the dashboard/services pages.

## Primary Service Labels

These labels should only be used on the primary service.

| Label | Required | Default | Description |
|-------|----------|---------|-------------|
| `lostack.primary` | Yes | - | Set this on the primary container that will be routed by Traefik |
| `lostack.port` | Yes | - | Primary service's web port |
| `lostack.enable` | Yes | - | Set this to true in all packages. This tells LoStack to list it in the services page |
| `lostack.autostart` | No | `true` | This should be true on packages that start/stop automatically. If a package should run 24/7, set this to false |
| `lostack.default_duration` | No | `5m` | The duration the package should stay up for without inactivity |
| `lostack.details` | No | - | Depot details text |
| `lostack.tags` | No | - | Searchable Depot Tags (comma-separated: `a,b,c`) |
| `lostack.project_url` | No | - | Project details URL (GitHub preferred) |
| `lostack.container_author` | No | - | Container image author |
| `lostack.package_author` | No | - | LoStack package author |

## General Labels

These labels can be used on every container.

| Label | Required | Description |
|-------|----------|-------------|
| `lostack.group` | Yes | Service group name. This should be the name of the primary container and is the prefix used for routing |
| `homepage.description` | No | Service description |
| `homepage.group` | Yes | Sorting group on automatic dashboard, separate from LoStack service launch group |
| `homepage.href` | No | This only populates GetHomePage; LoStack will ignore this value |
| `homepage.icon` | No | Depot, dashboard, and container manager icon |
| `homepage.name` | Yes | Friendly name of service in depot and on dashboard, as well as in GetHomePage |

## Example Configuration

```yaml
labels:
  - lostack.primary=true
  - lostack.port=80
  - lostack.enable=true
  - lostack.group=code-server
  - homepage.name=Code Server
  - homepage.group=Apps
  - homepage.icon=vscode
  - homepage.href=https://code-server.${DOMAINNAME}/
```