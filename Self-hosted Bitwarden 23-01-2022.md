# Self-hosted Bitwarden with Docker Compose
We are going to install a Bitwarden server that runs behind the [Linuxserver.io SWAG Reverse Proxy](https://www.linuxserver.io/blog/2020-08-26-setting-up-authelia). The Bitwarden server is only accessable over SSL and authentication is handled by [Authelia](https://www.authelia.com/) and use of [Yubico Yubikey](https://www.yubico.com/) is possible.

## Reading the docs
[Install and deploy Bitwarden on Linux](https://bitwarden.com/help/install-on-premise-linux/)

## System specifications
|                | Minimum                        | Recommended                  |
| :-             | :----------------------------: | :-----------:                |
| Processor      | x64, 1.4GHz                    | x64, 2GHZ Dual Core          |
| Memory         | 2GB RAM                        | 4GB RAM                      |
| Storage        | 10GB                           | 25GB                         |
| Docker Version | Engine 19+ and Compose 1.24+   | Engine 19+ and Compose 1.24+ |

## What we want
We want to run a self-hosted Bitwarden instance running in Docker containers and managed with Docker Compose. The Bitwarden instance needs to be accessable only over SSL configured behind the [SWAG](https://www.linuxserver.io/blog/2020-08-26-setting-up-authelia) Nginx reverse proxy from [Linuxserver.io](https://linuxserver.io).

Besides password- and secretmanagement we also want the API being available so we can automate secret management in our automation pipelines without the use of a thirth-party company like [Hashicorp Vaul](https://www.vaultproject.io/) or [Cyberark Conjur](https://www.conjur.org/).

Cloud services are nice, but don't trust them blind with your stuff, especially when it's about your secrets. For example []Lastpass](https://www.bleepingcomputer.com/news/security/lastpass-users-warned-their-master-passwords-are-compromised/) that warned their users that their **Master Password** is compromised.

In the end we want our Bitwarden instance secured with a hardware token like the [**Yubico Yubikey**](https://www.yubico.com/)

## Our goal
* Get your secrets out of the cloud
* Migrate your secrets to self-hosted Bitwarden
* Run Bitwarden in Docker with Docker Compose
* Ensure SSL with SWAG Reverse Proxy
* Enabled secret management with Bitwarden API for CI/CD automation
* Secured with Yubikey hardware token

## Lets start
When reading the Bitwarden documentation is quit fast clear to me that the proposed installation of Bitwarden won't fit the needs that I have. Since I'm running a big bad mofo of a Docker Server with my own architecture we are going to tweak some shizzle to fit my needs.

### Part 1 - Install Bitwarden Password Manager
Bitwarden works and SSL ensured by SWAG Reverse Proxy server.

* Be sure that the environment is fully updated and you have the correct Docker and Docker Compose versions
* Configure SWAG Docker Compose file by adding the sub-domain for Bitwarden
* Configure reverse proxy example that comes with SWAG by renameing `bitwarden.subdomain.conf.sample` to `bitwarden.subdomain.conf`
* Edit `bitwarden.subdomain.conf` and change the upstream app from `bitwarden` to `bitwarden-nginx`
__note: probably renaming bitwarden.subdomain.conf.sample to bitwarden.subdomain.conf will be enough__
* Restart SWAG
* Prepare data directory on the docker host by creating directory and setting the proper permissions
* Get the Bitwarden installation script from [here](curl -Lso bitwarden.sh https://go.btwrdn.co/bw-sh) and make the script executable
* Run the script, and use the following values;
1. Domain name for instance; `vault.example.com`
2. Don't generate a free Let's Encrypt certificate. This will not work since you are behind a proxy. SWAG is going to handle this part
3. Database name **vault** which is proposed as default (probably because of the domain name)
4. Go to [https://bitwarden.com/host] to get a Installation ID and Installation Key
5. You don't have a SSL certificate, so press **n**
6. You do want to generate a self-signed SSL certificate for end-2-end SSL
7. Ignore the warning about the self-signed certificate since we are going to solve that with proxy server
8. Installation script complete but with files on locations where I don't want them
9. In the **bwdata/docker** directory you'll find the **docker-compose.yml** file. Here you need to change the paths of the volumes. In my case I'll move the paths configured in **volumes** to `/docker_data/bitwarden`
10. The **.env** files I keep in `/docker_files/bitwarden`
11. Set Docker Compose version to **version 2** so we can add some resource limits later
12. My SWAG server uses a network called **proxy** so configure the docker-compose file with this proxy network, but try only to configure the containers that needed to be accessable from the outside. Basically you can replace `public` to `proxy`
13. Remove `ports` configuration from Nginx service in the `docker-compose.yml`
__If you leave the ports overthere the server won't start because we changed the docker-composer version to use to 2. We don't need them since the reverse proxy server is able to access the Bitwarden webserver via the defined proxy netwerk. So, no reason to expose and make the server potencially vulnerable for attacks__
14. Configure the email settings in `global.override.env`

### Part 2 - Harden security by enabling Authelia, Duo and Yubikey usage
Hardening Bitwarden by enabling authentication via Authelia, Duio and Yubikey Hardware token.

* Ensure Bitwarden works
* Configure Bitwarden reverse proxy configuration to enable Authelia
* Configure Duo
* Configure Yubikey

### Part 3 - Enable use of Bitwarden API for secret automation
Enable Bitwarden API for secret automation and usage from CI/CD pipelines

* Configure API to accept connections over secure API tokens in reverse proxy configuration
* Configures Gitlab test pipeline

### Part 4 - Set memory limits on the bitwarden containers
* Let the environment run for a while
* Monitor the system usage of the Bitwarden containers with `docker stats`
* Ensure you use **version 2** in the docker-compose.yml
* Configure Docker Compose file with `mem_limit` and `mem_reservation`

### Part 5 - Celebrate victory with some beer and a cigar
Be awesome!


### Installation step
1. Ensure you have Docker and Docker Compose installed and is the correct version
2. Your domain is configured.
__note: We are going to use https://vault.itmagix.nl for our instance. The IT MAG!X domain is configured with a wildcard that points to the SWAG reverse proxy server__
3. Configure the SWAG Docker Compose file to accept your Bitwarden sub-domain and a SSL certificate is generated
4. Configure a reverse proxy in the Swag Nginx proxy-confs
__note: Bitwarden is not a member of the [Linuxserver.io](https://linuxserver.io) image family, but they do provide a Reverse Proxy configuration in SWAG. Thanks for that!__
5. 

