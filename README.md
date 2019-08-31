# ApacheDS
This is a clone of openmicroscopy/apacheds-docker.

This Docker image provides an [ApacheDS](https://directory.apache.org/apacheds/) LDAP server. Optionally it could be used to provide a [Kerberos server](https://directory.apache.org/apacheds/advanced-ug/2.1-config-description.html#kerberos-server) as well.

The project sources can be found on [GitHub](https://github.com/openmicroscopy/apacheds-docker). The Docker image on [Docker Hub](https://registry.hub.docker.com/u/openmicroscopy/apacheds/).


## Build

    git clone https://github.com/clonm/apacheds-docker.git
    cd apacheds-docker
    docker build -t clonm/apacheds .


## Installation

The folder */var/lib/apacheds* contains the runtime data and thus has been defined as a volume. The image uses exactly the file system structure defined by the [ApacheDS documentation](https://directory.apache.org/apacheds/advanced-ug/2.2.1-debian-instance-layout.html).

The container can be started issuing the following command:

    docker run --name ldap -d -p 389:10389 clonm/apacheds

There is also a docker-compose file that does the same thing

    docker-compose up

(use `-d` to start in detached mode)

## Usage

You can manage the ldap server with the admin user *uid=admin,ou=system* and the default password *secret*. The *default* instance comes with a pre-configured partition *dc=cloyne,dc=org*.

An individual admin password should be set following [this manual](https://directory.apache.org/apacheds/basic-ug/1.4.2-changing-admin-password.html).

Then (from inside the container) you can import entries into that partition via
your own *ldif* file:

    ldapadd -v -h <your-docker-ip>:389 -c -x -D uid=admin,ou=system -w <your-admin-password> -f sample.ldif

## Testing

For testing, you can poke around with [Apache Directory
Studio](https://directory.apache.org/studio/) as an LDAP client. After
downloading and installing,
1. run the server container with `docker-compose up`
2. In ApacheDirectoryStudio, go to the *LDAP* menu -> *New Connection...*
    1. In *Connection Name*, put whatever you want, e.g. *ldap_test*
    1. For *Hostname*, put `localhost`
    1. Leave the rest of the settings as default and Click *Check Network
       Parameter*. It should say "The connection was
       established successfully".
    1. Click *Next >* For *Bind DN or user*, put `uid=admin,ou=system`
    1. For *Bind password*, put `secret`
    1. Click *Check Network Parameter*. It should say "The authentication was
       successful".

### Troubleshooting
* If you get `ERR_04110_CANNOT_CONNECT_TO_SERVER`, you probably forgot to start
   the server. Use `docker ps` to check what containers are running, and restart
   if necessary.
* If you get `ERR_04169_RESPONSE_QUEUE_EMPTIED`: the *Bind DN or user* parameter
    is wrong. It should be `uid=admin,ou=system`.
* If you get `ERR_04160_SESSION_HAS_BEEN_CLOSED`: the *Bind DN or user* parameter
    is wrong. It should be `uid=admin,ou=system`.


## Customization

It is also possible to start up your own defined Apache DS *instance* with your own configuration for *partitions* and *services*. Therefore you need to mount your [config.ldif](https://github.com/openmicroscopy/apacheds-docker/blob/master/instance/config.ldif) file and set the *APACHEDS_INSTANCE* environment variable properly. In the provided sample configuration the instance is named *default*. Assuming your custom instance is called *yourinstance* the following command will do the trick:

    docker run --name ldap -d -p 389:10389 -e APACHEDS_INSTANCE=yourinstance -v /path/to/your/config.ldif:/bootstrap/conf/config.ldif:ro clonm/apacheds


It would be possible to use this ApacheDS image to provide a [Kerberos server](https://directory.apache.org/apacheds/advanced-ug/2.1-config-description.html#kerberos-server) as well. Just provide your own *config.ldif* file for that. Don't forget to expose the right port, then.

Also other services are possible. For further information read the [configuration documentation](https://directory.apache.org/apacheds/advanced-ug/2.1-config-description.html).

