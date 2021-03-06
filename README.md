# Wordpress Framework

Wordpress framework build on Docker, Wercker and Dokku for deployment

## Local Development

### Prerequisites

You must have Docker and Wercker Cli installed before beginning.

### Getting Started

* Clone this repo
* cd into the root of the repo and run `tools/dev`

The application should now be running on port 80. You will be attached to the running container and the log viewer will run.

If you need to run adhoc commands, hit `ctrl+c` to exit the log viewer. You can return to the log viewer by simply running `log`

To exit fully from the container, hit `ctrl+c` to exit the log viewer and then type `exit` to exit the container.

## Deployment

The framework is setup to deploy to a Dokku instance using Wercker.

### Wercker Project Setup

1. Navigate to your project and open up `Settings > SSH Keys`
2. Create a new SSH key
3. Open up `Settings > Targets`
4. Create a new target and give it a name *(production, staging etc.)*
5. Create three environment variables:
    - **DOKKU_KEY** - SSH Key - [key created in step 2]
    - **DOKKU_HOST** - String - [your dokku host ip or fqdn]
    - **DOKKU_APP** - String - wordpress
6. Save the target settings

### Dokku Server Setup

Once you've booted your Dokku server, on the installation screen, copy and paste the public SSH key from the Wercker Project target. Then run the following steps:

```bash
# Set some environment variables
export MARIADB_IMAGE_VERSION="10.1.13"
export DOKKU_DATA_DIR="/var/lib/dokku/data"

# Install MariaDB Dokku plugin
dokku plugin:install https://github.com/dokku/dokku-mariadb.git

# Create our Wordpress app
dokku apps:create wordpress

# Create our MariaDB service and link to our Wordpress app
dokku mariadb:create wordpress-db
dokku mariadb:link wordpress-db wordpress
dokku mariadb:info wordpress-db

# Set our config values, obtained from the last command
dokku config:set wordpress WORDPRESS_DB_HOST=dokku-mariadb-wordpress-db
dokku config:set wordpress WORDPRESS_DB_NAME=wordpress-db
dokku config:set wordpress WORDPRESS_DB_USER=mariadb
dokku config:set wordpress WORDPRESS_DB_PASSWORD=[PASSWORD]
dokku config wordpress

# Create our storage directory and mount to our Wordpress app
mkdir -p "$DOKKU_DATA_DIR/wordpress/wp-content"
dokku storage:mount wordpress "$DOKKU_DATA_DIR/wordpress/wp-content:/var/www/html/wp-content"
```
