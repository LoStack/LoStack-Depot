# LoStack Depot

This repo serves as the backend for the LoStack Depot. It is designed to be compatible with GetHomePage and should work with DockGE.

Pull Requests are Welcome!

---

## Package Standard

LoStack implements its own labeling standard to replace Traefik's routing. There may be certain instances where it is appropriate to use Traefik labeling, but the default LoStack routing labels should be used wherever possible.

LoStack borrows GetHomePage's labeling system (and icon system), which means packages installed with LoStack will appear on GetHomePage with the correct labels and description. LoStack's built-in Dashboard is lacking in comparison to GetHomePage. A first-party plugin to generate GetHomePage dashboards per-user is in the works.

LoStack reads the labels from running containers and automatically populates the dashboard/services pages.

### Primary Service Labels

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
| `lostack.force_disable_autostart` | No | `false` | Disables autostart option in LoStack Services menu |
| `lostack.force_disable_access_control` | No | `false` | Disables access_control option in LoStack Services menu |
| `lostack.force_disable_autoupdate` | No | `false` | Disables auto_update option in LoStack Services menu (NYI) |
| `lostack.force_compose_edit` | No | `false` | Disables the editor page for the service |
| `lostack.root` | No | `false` | Mounts a container to the domain root, only one service may have this enabled |


### General Labels

These labels can be used on every container.

| Label | Required | Description |
|-------|----------|-------------|
| `lostack.group` | Yes | Service group name. This should be the name of the primary container and is the prefix used for routing |
| `homepage.description` | No | Service description |
| `homepage.group` | Yes | Sorting group on automatic dashboard, separate from LoStack service launch group |
| `homepage.href` | No | This only populates GetHomePage; LoStack will ignore this value |
| `homepage.icon` | No | Depot, dashboard, and container manager icon |
| `homepage.name` | Yes | Friendly name of service in depot and on dashboard, as well as in GetHomePage |


### Compose Structure

**UPCOMING CHANGE**: LoStack will automatically generate bridge networks in the future. At that time, packages will be updated to remove the default traefik_network. 

LoStack uses a limited version of the Docker Compose standard for its package system
Currently, only the services section of Docker Compose is supported
Networks and volumes will be ignored if supplied in a package. 

LoStack recommends a consistent naming scheme, while you don't have to name them like this, it makes managment easier.
If you wish to submit a package to LoStack, you must follow the naming format for the service and child container.
Abstract naming schemes like "app" for the primary container or "db" are not allowed to prevent naming conflicts between containers.

The primary container should be the same as the package name, this is also the subdomain the package will be served on. Container names should be lowercase.

See below for an example container, with a primary service and supporting db:
```yaml
services:
  ackee:
    image: electerious/ackee
    container_name: ackee
    hostname: ackee
    networks:
     - traefik_network
    environment:
      - WAIT_HOSTS=ackee-mongo:27017
      - ACKEE_MONGODB=mongodb://ackee-mongo:27017/ackee
      - ACKEE_USERNAME=${LDAP_ADMIN_USERNAME}
      - ACKEE_PASSWORD=${ADMIN_PASSWORD}
      - ACKEE_TTL=3600000
      - ACKEE_ALLOW_ORIGIN=https://ackee.${DOMAINNAME}
    depends_on:
      - ackee-mongo
    restart: always  
    labels:
      - "lostack.enable=true"
      - lostack.port=3000
      - "homepage.name=Ackee"
      - "homepage.group=Apps"
      - "homepage.icon=mdi-google-analytics"
      - "homepage.description=Analytics Tool"
      - "homepage.href=https://ackee.${DOMAINNAME}/"
      - lostack.autostart=true
      - lostack.group=ackee
      - lostack.default_duration=15m
      - lostack.primary=true
      - "lostack.details=Self-hosted, Node.js based analytics tool for those who care about privacy."
      - lostack.tags=analytics,privacy,api,graphql
      - lostack.project_url=https://github.com/electerious/Ackee

  ackee-mongo:
    image: mongo
    container_name: ackee-mongo
    hostname: ackee-mongo
    networks:
     - traefik_network
    volumes:
      - ${APPDATA_DIR}/ackee/db:/data/db
    labels:
      - "homepage.name=Ackee MongoDB"
      - "homepage.group=Databases"
      - "homepage.icon=mongodb"
      - "homepage.description=Ackee Database"
      - "lostack.autostart=true"
      - "lostack.group=ackee"
      - "lostack.enable=true"
```


### Environment Variables
LoStack uses a set of standardized environment variables to make depot deployment easy.
You may use any of these variables in a LoStack package.

```properties
# # # # HOST # # # #
HOSTNAME
DOMEXT
DOMAINNAME
HOST_IP
DNS_IP
TRUSTED_PROXYS
TZ


# # # # PASSWORDS # # # #
ADMIN_PASSWORD
DATABASE_PASSWORD


# # # # VOLUMES # # # #
APPDATA_DIR
CERTS_DIR
CONFIG_DIR
LOGS_DIR
MEDIA_DIR
ENV_DIR
DOCKER_SOCKET


# # # # PORTS # # # #
TRAEFIK_PORT_HTTP
TRAEFIK_PORT_HTTPS
OPENLDAP_PORT
OPENLDAP_LDAPS_PORT


# # # # OPENLDAP # # # #
LDAP_ADMIN_GROUP
LDAP_ADMIN_PASSWORD
LDAP_DEFAULT_GROUP
LDAP_CONFIG_PASSWORD
LDAP_RFC2307BIS_SCHEMA
LDAP_ORGANISATION
LDAP_BASE_DN
LDAP_ADMIN_USER
```


### Standardized Media Folders
LoStack uses an opinionated media folder structure by default
You can use these folders in your packages
```properties
# # # # MEDIA DIRS # # # #
./${MEDIA_DIR}/audiobooks
./${MEDIA_DIR}/books
./${MEDIA_DIR}/comics
./${MEDIA_DIR}/documents
./${MEDIA_DIR}/downloads
./${MEDIA_DIR}/images
./${MEDIA_DIR}/manga
./${MEDIA_DIR}/models
./${MEDIA_DIR}/movies
./${MEDIA_DIR}/music
./${MEDIA_DIR}/podcasts
./${MEDIA_DIR}/recordings
./${MEDIA_DIR}/tv
./${MEDIA_DIR}/www
./${MEDIA_DIR}/youtube
./${MEDIA_DIR}/youtube/general
./${MEDIA_DIR}/youtube/music
./${MEDIA_DIR}/youtube/podcasts
./${MEDIA_DIR}/youtube/temp

# # # # DOWNLOADS # # # #
./${MEDIA_DIR}/downloads/audiobooks
./${MEDIA_DIR}/downloads/books
./${MEDIA_DIR}/downloads/comics
./${MEDIA_DIR}/downloads/documents
./${MEDIA_DIR}/downloads/general
./${MEDIA_DIR}/downloads/images
./${MEDIA_DIR}/downloads/manga
./${MEDIA_DIR}/downloads/models
./${MEDIA_DIR}/downloads/movies
./${MEDIA_DIR}/downloads/music
./${MEDIA_DIR}/downloads/podcasts
./${MEDIA_DIR}/downloads/recordings
./${MEDIA_DIR}/downloads/temp
./${MEDIA_DIR}/downloads/tv
./${MEDIA_DIR}/downloads/www
./${MEDIA_DIR}/downloads/youtube
./${MEDIA_DIR}/downloads/youtube/general
./${MEDIA_DIR}/downloads/youtube/music
./${MEDIA_DIR}/downloads/youtube/podcasts
./${MEDIA_DIR}/downloads/youtube/temp
```