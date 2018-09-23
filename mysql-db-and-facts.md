# Creating MySQL Database using Ansible

## Sample Playbooks

Sample playbooks are available in our samples repo here:

https://github.com/Azure-Samples/ansible-playbooks

## Prerequisites

This sample requires Ansible 2.7.

It also requires that MySQL client is installed on your local system. This could be done for instance as follows (on Ubuntu):

```
sudo apt-get install mysql-client
```
## What will we do in this sample

- Create MySQL Server
- Create database
- Query servers in given resource groups
- Query databases on a single MySQL Server
- Create MySQL firewall rule to access MySQL Server from outside
- List all tables for all databases on a single MySQL Server

This sample illustrates how to create  MySQL database resources.
It also shows how to use new facts modules with new flattened and simplified format.

## Create a resource group
A resource group is a logical container into which Azure resources are deployed and managed.  

The following example creates a resource group named *myResourceGroupSQL* in the *eastus* location.

``` yaml
- hosts: localhost
  vars:
    resource_group: myResourceGroupSQL
    location: eastus 
  tasks:
    - name: Create a resource group
      azure_rm_resourcegroup:
        name: "{{ resource_group }}"
        location: "{{ location }}"
```


## Creating SQL Server and Database

Following example creates MySQL server and database in myResourceGroupSQL:

``` yaml
- hosts: localhost
  vars:
    resource_group: myResourceGroupSQL
    location: eastus
    mysqlserver_name: mysql{{ rpfx }}
    mysqldb_name: sqldbtest
    admin_username: admxyz
    admin_password: Abcpasswordxyz12!
  tasks:
    - name: Create MySQL Server
      azure_rm_mysqlserver:
        resource_group: "{{ resource_group }}"
        name: "{{ mysqlserver_name }}"
        sku:
          name: B_Gen5_1
          tier: Basic
        location: "{{ location }}"
        version: 5.6
        enforce_ssl: no
        admin_username: "{{ admin_username }}"
        admin_password: "{{ admin_password }}"
        storage_mb: 51200
    - name: Create instance of MySQL Database
      azure_rm_mysqldatabase:
        resource_group: "{{ resource_group }}"
        server_name: "{{ mysqlserver_name }}"
        name: "{{ mysqldb_name }}"
```

## Configure firewall rule

A server-level firewall rule allows an external application, such as the **mysql** command-line tool or MySQL Workbench to connect to your server through the Azure MySQL service firewall. 
The following example creates a firewall rule called **extenalaccess** that allows connections from any external IP address. Substitute **startIpAddress** and **endIpAddress** with range of IP addresses that correspond to where you'll be connecting from. 

``` yaml
- hosts: localhost
  vars:
    resource_group: myResourceGroupSQL
    mysqlserver_name: mysql{{ rpfx }}
  tasks:
    - name: Open firewall to access MySQL Server from outside
      azure_rm_resource:
        api_version: '2017-12-01'
        resource_group: "{{ resource_group }}"
        provider: dbformysql
        resource_type: servers
        resource_name: "{{ os.servers[0].name }}"
        subresource:
          - type: firewallrules
            name: externalaccess
        body:
          properties: 
            startIpAddress: "0.0.0.0"
            endIpAddress: "255.255.255.255"
```

Note: We are using **azure_rm_resource** module to perform this task, which allows direct use of REST API, as appropriate module is not yet available in Ansible 2.7.

Note: Connections to Azure Database for MySQL communicate over port 3306. If you try to connect from within a corporate network, outbound traffic over port 3306 might not be allowed. If this is the case, you can't connect to your server unless your IT department opens port 3306.


## Using facts to query MySQL Servers in current resoource group

Following tasks will query all the servers in specified resource group and dump teh output:

``` yaml
  - name: Query MySQL Servers in current resource group
    azure_rm_mysqlserver_facts:
      resource_group: "{{ resource_group }}"
    register: os

  - name: Dump MySQL Server facts
    debug:
      var: os
```

Please note that output is very simple and contains only necessary fields. In addition it's very similar to the main module input:

``` json
  "servers": [
      {
          "admin_username": "admxyz",
          "enforce_ssl": true,
          "fully_qualified_domain_name": "mysql46.mysql.database.azure.com",
          "id": "/subscriptions/685ba005-af8d-4b04-8f16-a7bf38b2eb5a/resourceGroups/zimspostgresrg/providers/Microsoft.DBforMySQL/servers/mysql46",
          "location": "eastus",
          "name": "mysql46",
          "resource_group": "zimspostgresrg",
          "sku": {
              "capacity": 1,
              "family": "Gen5",
              "name": "B_Gen5_1",
              "tier": "Basic"
          },
          "storage_mb": 5120,
          "tags": null,
          "user_visible_state": "Ready",
          "version": "5.6"
      }
  ]
```

## Using facts module to query all the databases on the server

Following tasks will query all the databases from the first server on the list and dump the output:

``` yaml
  - name: Query MySQL Databases
    azure_rm_mysqldatabase_facts:
      resource_group: "{{ resource_group }}"
      server_name: "{{ os.servers[0].name }}"
    register: do

  - name: Dump MySQL Database Facts
    debug:
      var: do
```

Please note output for databases:

``` json
  "databases": [
      {
          "charset": "utf8",
          "collation": "utf8_general_ci",
          "name": "information_schema",
          "resource_group": "zimspostgresrg",
          "server_name": "mysql46"
      },
      {
          "charset": "latin1",
          "collation": "latin1_swedish_ci",
          "name": "mysql",
          "resource_group": "zimspostgresrg",
          "server_name": "mysql46"
      },
      {
          "charset": "utf8",
          "collation": "utf8_general_ci",
          "name": "performance_schema",
          "resource_group": "zimspostgresrg",
          "server_name": "mysql46"
      },
      {
          "charset": "latin1",
          "collation": "latin1_swedish_ci",
          "name": "sqldbtest",
          "resource_group": "zimspostgresrg",
          "server_name": "mysql46"
      }
  ]
```

## Testing access to MySQL server

Here we will just call **mysql** command (as specified in prerequisites).

Following task simply iterates over list of the databases acquited previously and queries list of tables for each of them.

This sample illustrates how to use MySQL Server / Database information obtained using appropriate facts modules

``` yaml
    - name: Dump tables
      shell: mysql --host={{ os.servers[0].fully_qualified_domain_name }} --user={{ os.servers[0].admin_username }}@{{ os.servers[0].name }} --password={{ admin_password }} --verbose {{ item.name }} -e "show tables"
      with_items: "{{ do.databases }}"
      register: output

    - debug:
        var: output
```



