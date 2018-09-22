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

## Creating SQL Server and Database

Use the sample called **mysql_create.yml**. Run:

```
ansible-playbook mysql_create.yml
```

First task creates SQL Server:

``` yaml
  - name: Create MySQL Server
    azure_rm_mysqlserver:
      resource_group: "{{ resource_group }}"
      name: "{{ mysqlserver_name }}"
      sku:
        name: B_Gen5_1
        tier: Basic
      location: "{{ location }}"
      version: 5.6
      enforce_ssl: True
      admin_username: "{{ admin_username }}"
      admin_password: "{{ admin_password }}"
      storage_mb: 51200
```

Second task creates a sample database in previously created SQL Server:

``` yaml
  - name: Create instance of MySQL Database
    azure_rm_mysqldatabase:
      resource_group: "{{ resource_group }}"
      server_name: "{{ mysqlserver_name }}"
      name: "{{ mysqldb_name }}"
```

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


## Using facts to query all the databases on the server

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


## Opening firewall

Before MySQL Server can be accessed, it's necessary to add appropriate firewall rule. As there's still no module to do this in Ansible 2.7, we will use **azure_rm_resource** module as follows:

``` yaml
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

Code above will give access to our MySQL Server from any IP Address.

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



