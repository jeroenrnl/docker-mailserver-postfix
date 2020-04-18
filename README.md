# docker-mailserver-postfix
This is a Docker image for a Postfix/Dovecot mailserver. The project is part of the 
[docker-mailserver](https://github.com/technicalguru/docker-mailserver) project but can run separately 
without the other components. However, a database server is always required to store structural data. 
E-Mails itself are stored on file system.

Related images:
* [docker-mailserver](https://github.com/technicalguru/docker-mailserver) - The main project, containing composition instructions
* [docker-mailserver-postfixadmin](https://github.com/technicalguru/docker-mailserver-postfixadmin) - Image for PostfixAdmin (Web UI to manage mailboxes and domain in Postfix)
* [docker-mailserver-amavis](https://github.com/technicalguru/docker-mailserver-amavis) - Amavis, ClamAV and SpamAssassin (provides spam and virus detection)
* [docker-mailserver-roundcube](https://github.com/technicalguru/docker-mailserver-roundcube) - Roundcube Webmailer

# Tags
The following versions are available from DockerHub. The image tag matches the Postfix version.

* [3.4.8-01, 3.4.8, 3.4, 3, latest](https://hub.docker.com/repository/docker/technicalguru/mailserver-postfix) - [Dockerfile](https://github.com/technicalguru/docker-mailserver-postfix/blob/3.4.8-01/Dockerfile)

# Features
* Bootstrap from scratch: See more information below.
* Standard SMTP and IMAP ports
* TLS encryption (optional)
* AntiVirus and AntiSpam integration (optional)

# License
_docker-mailserver-postfix_  is licensed under [GNU LGPL 3.0](LICENSE.md). As with all Docker images, these likely also contain other software which may be under other licenses (such as Bash, etc from the base distribution, along with any direct or indirect dependencies of the primary software being contained).

As for any pre-built image usage, it is the image user's responsibility to ensure that any use of this image complies with any relevant licenses for all software contained within.

# Prerequisites
The following components must be available at runtime:
* [MySQL >8.0](https://hub.docker.com/\_/mysql) or [MariaDB >10.4](https://hub.docker.com/\_/mariadb) - used as database backend for domains and mailboxes. 

# Usage

## Environment Variables
_mailserver-postfix_  requires various environment variables to be set. The container startup will fail when the setup is incomplete.

| **Variable** | **Description** | **Default Value** |
|------------|---------------|-----------------|
| `PF_SETUP_PASS` | The password of the database administrator (`root`). This value is required for the initial bootstrap only in order to setup the database structure. It can and shall be removed after successful setup. |  |
| `PF_DB_HOST` | The hostname or IP address of the database server | `localhost` |
| `PF_DB_USER` | The name of the database user. **Attention!** You shall not use an administrator account. | `postfix` |
| `PF_DB_PASS` | The password of the database user | `password` |
| `PF_DB_NAME` | The name of the database | `postfix` |
| `PF_MYDOMAIN` | The first and primary mail domain of this server. Postfix requires this for setup but you can configure multiple main domains. | `localdomain` |
| `PF_MYHOSTNAME` | The hostname that Postfix uses to greet clients. | (name of host) |
| `PF_MYORIGIN` | The domain to be used for local mails (usually name of host). | value of `PF_MYHOSTNAME` |
| `PF_AMAVIS_SERVICE_NAME` | The hostname or IP address of an Amavis instance in order to fight spam and viruses. No AntiSpam and AntiVirus detection takes place when left empty |  |
| `PF_AMAVIS_SERVICE_PORT` | The port of the Amavis instance. | `10024` |
| `PF_TLS_CERT_FILE` | SSL server certificate for TLS. | `/etc/ssl/certs/ssl-cert-snakeoil.pem` |
| `PF_TLS_KEY_FILE` | Key file for SSL server certificate. | `/etc/ssl/certs/ssl-cert-snakeoil.key` |
| `PF_TLS_CAPATH` | Directory that contains trusted CA root certificates. | `/etc/ssl/certs` |
| `PF_TLS_CAFILE` | Name of single file that contains trusted CA root certificates. | `/etc/postfix/CAcert.pem` |

## Volumes
You need to provide a data volume in order to secure your mailboxes from data loss. Map the volume to `/var/vmails` folder inside the container.

Additional volumes are required to map your TLS certificate into the container.

## Ports
_docker-mailserver-postfix_  exposes 5 ports by default:
* Port 25 - the traditional SMTP port. This port must be accessible from other hosts to send e-mails to you.
* Port 587 - the default port nowadays for SMTP. Still, some mail providers do not support them. This port shall be accessible from other hosts.
* Port 143 - the default port for SMTP authentication and IMAP mail access. This port must be accessible for your mail agents, e.g. Outlook or Thunderbird.
* Port 993 - the port for incoming e-mails using IMAP protocol. This port must be accessible for your mail agents, e.g. Outlook or Thunderbird.
* Port 10025 - a local SMTP delivery port for mails that were checked from Amavis. **Attention!** You need to make sure that this port is not accessible by any other host than your Amavis service because it is not protected and can be used for SPAM attacks.
 
## Running the Container
The [main mailserver project](https://github.com/technicalguru/docker-mailserver) has examples of container configurations:
* [with docker-compose](https://github.com/technicalguru/docker-mailserver/tree/master/examples/docker-compose)
* [with Kubernetes YAML files](https://github.com/technicalguru/docker-mailserver/tree/master/examples/kubernetes)
* [with HELM charts](https://github.com/technicalguru/docker-mailserver/tree/master/helm-charts)

## Bootstrap and Setup
Once you have started your Postfix container successfully, it is now time to perform the first-time setup for your mailserver. It is highly recommended to use [docker-mailserver-postfixadmin](https://github.com/technicalguru/docker-mailserver-postfixadmin) for this purpose. However, you can use your own [PostfixAdmin](https://github.com/postfixadmin/postfixadmin) installation.

1. Create your PostfixAdmin administrator account (see [docker-mailserver-postfixadmin](https://github.com/technicalguru/docker-mailserver-postfixadmin/blob/master/README.md))
1. Create your primary domain matching the environment variable `PF_MYDOMAIN`
1. Create your first mailbox in this domain

# TLS Configuration
Only two environment variables are required in order to secure your mailserver by TLS. `PF_TLS_CERT_FILE` and `PF_TLS_KEY_FILE` will ensure that mails can be sent to you in a secure way. However, bear in mind that these certificates expire and that there is currently no automatic check available to warn you about the expiration of this certificate. As these variables hold path names only, it is required to map your certificate files into the running container using volumes.

You'll need to issue `postconfig reload` after you've changed the certificate. 

# Additional Postfix/Dovecot customization
You can further customize `main.cf`, `master.cf` and other Postfix configuration files. Please follow these instructions:

1. Check the `/usr/local/mailserver/templates` folder for already existing customizations. 
1. If you configuration file is not present yet, take a copy of the file from `/etc/postfix` folder.
1. Customize your Postfix and/or Dovecot configuration file.
1. Provide your customized file(s) back into the appropriate template folder at `/usr/local/mailserver/templates` by using volume mappings.
1. (Re)Start the container. If you configuration was not copied correctly then log into the container (bash is available) and issue `/usr/local/mailserver/reset-server.sh`. Then restart again.

# Issues
This Docker image is mature and replaced my own mailserver in production. However, several issues are still unresolved:

* [#1](https://github.com/technicalguru/docker-mailserver-postfix/issues/1) - Logging to stdout is showing Postfix log only. Dovecot and other logs are still not showing up.
* [#2](https://github.com/technicalguru/docker-mailserver-postfix/issues/2) - DKIM support is missing
* [#3](https://github.com/technicalguru/docker-mailserver-postfix/issues/3) - SPF support is missing

# Contribution
Report a bug, request an enhancement or pull request at the [GitHub Issue Tracker](https://github.com/technicalguru/docker-mailserver-postfix/issues).
