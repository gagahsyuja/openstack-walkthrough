## Keystone - Identity Service
The Identity service is the first service a user interacts with. Once authenticated, and end user can use their identity to access other OpenStack services. Here are the steps to get keystone up and running:
1. [Prequisites](#prequisites)
2. [Install and Configure Components](#install-and-configure-components)
3. [Finalize The Installation](#finalize-the-installation)
4. [Verify Operation](#verify-operation)

### Prequisites
To get started, create a database for keystone to store its data:
```bash
# mysql
```
Create the **keystone** database:
```bash
MariaDB [(none)]> CREATE DATABASE keystone;
```
Grant proper access to the **keystone** database:
```bash
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' \
IDENTIFIED BY 'KEYSTONE_DBPASS';

MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY 'KEYSTONE_DBPASS';
```
Replace **KEYSTONE_DBPASS** with a suitable password.

You can exit the database client after finishing steps above.

### Install and Configure Components
Next, run the following commands to install the keystone package:
```bash
# apt install keystone
```
Edit `/etc/keystone/keystone.conf` file and complete these steps:

- In the **[database]** section, configure database access:

    ```bash
    [database]
    # ...
    connection = mysql+pymysql://keystone:KEYSTONE_DBPASS@controller/keystone
    ```
    Replace **KEYSTONE_DBPASS** with the password you chose for the database.

- In the **[token]** section, configure the Fernet token provider:

    ```bash
    [token]
    # ...
    provider = fernet
    ```

- Populate the Identity service database:
    
    ```bash
    # su -s /bin/sh -c "keystone-manage db_sync" keystone
    ```
- Initialize Fernet key repositories:

    ```bash
    # keystone-manage fernet_setup --keystone-user keystone --keystone-group keystone
    # keystone-manage credential_setup --keystone-user keystone --keystone-group keystone
    ```
- Bootstrap the identity service:
    ```bash
    # keystone-manage bootstrap --bootstrap-password ADMIN_PASS \
      --bootstrap-admin-url http://controller:5000/v3/ \
      --bootstrap-internal-url http://controller:5000/v3/ \
      --bootstrap-public-url http://controller:5000/v3/ \
      --bootstrap-region-id RegionOne
    ```
    Replace **ADMIN_PASS** with a suitable password.

Now, configure the Apache HTTP server config `/etc/apache2/apache2.conf` to modify the **ServerName** option to reference the controller node:
```bash
ServerName controller
```

### Finalize The Installation
To finalize the installation, restart the Apache service:
```bash
# systemctl restart apache2
```

### Verify Operation
Verify operation of the Identity service before installing other services.
1. Unset the temporary OS_AUTH_URL and OS_PASSWORD environment variable:
    ```bash
    $ unset OS_AUTH_URL OS_PASSWORD
    ```
2. As the **admin** user, request an authenticated token:
    ```bash
    $ openstack --os-auth-url http://controller:5000/v3 \
      --os-project-domain-name Default --os-user-domain-name Default \
      --os-project-name admin --os-username admin token issue

    Password:
    +------------+-----------------------------------------------------------------+
    | Field      | Value                                                           |
    +------------+-----------------------------------------------------------------+
    | expires    | 2016-02-12T20:14:07.056119Z                                     |
    | id         | gAAAAABWvi7_B8kKQD9wdXac8MoZiQldmjEO643d-e_j-XXq9AmIegIbA7UHGPv |
    |            | atnN21qtOMjCFWX7BReJEQnVOAj3nclRQgAYRsfSU_MrsuWb4EDtnjU7HEpoBb4 |
    |            | o6ozsA_NmFWEpLeKy0uNn_WeKbAhYygrsmQGA49dclHVnz-OMVLiyM9ws       |
    | project_id | 343d245e850143a096806dfaefa9afdc                                |
    | user_id    | ac3377633149401296f6c0d92d79dc16                                |
    +------------+-----------------------------------------------------------------+
    ```

That's it, **keystone** should be up and running!
