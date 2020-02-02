# teiid-mysql-rest

Guide to setup Teiid in standalone mode and create rest api to query from mysql database.

## Prerequisite

* Java 8
* Mysql

## Installation

* Install Java 8 and mysql
* Download latest version of Teiid from http://teiid.io/legacy/downloads_9x/ (With WildFly/Console) and extract the zip file.
* Download and extract the wildfly mysql connector
```bash
sudo wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-8.0.17.tar.gz

sudo tar -xvzf mysql-connector-java-8.0.17.tar.gz
```
* Create a directory inside downloaded wildfly to move the mysql connector (Here \<jboss-install> is the directory where Wildfly is extracted).
```bash
sudo mkdir -p <jboss-install>/module/system/layers/base/com/mysql/main
```
* copy the ` mysql-connector-java-8.0.17.jar` file to `<jboss-install>/module/system/layers/base/com/mysql/main` folder.
* Copy [module.xml](module.xml) fie to `<jboss-install>/module/system/layers/base/com/mysql/main` folder.
* Add driver and datasource to `standalone.xml` file in `<jboss-install>/standalone/configuration`
```xml
<driver name="mysql" module="com.mysql">
    <driver-class>com.mysql.cj.jdbc.Driver</driver-class>
    <xa-datasource-class>com.mysql.cj.jdbc.MysqlXADataSource</xa-datasource-class>
</driver>
```
```xml
<datasource jndi-name="java:jboss/datasources/mysql-ds" pool-name="mysqlDS" enabled="true">
  <connection-url>jdbc:mysql://localhost:3306/myDB</connection-url>
  <driver>mysql</driver>
  <transaction-isolation>TRANSACTION_READ_COMMITTED</transaction-isolation>
  <pool>
      <min-pool-size>10</min-pool-size>
      <max-pool-size>100</max-pool-size>
      <prefill>true</prefill>
  </pool>
  <security>
      <user-name>root</user-name>
      <password>root</password>
  </security>
  <statement>
      <prepared-statement-cache-size>32</prepared-statement-cache-size>
      <share-prepared-statements>true</share-prepared-statements>
  </statement>
</datasource>
```
( Update datasource with your mysql username and password )

* To Install Teiid using CLI script, run
```bash
<jboss-install>/bin/standalone.sh
```
Then in a separate console window execute

```bash
<jboss-install>/bin/jboss-cli.sh --file=bin/scripts/teiid-standalone-mode-install.cli
```
This will install Teiid subsystem into the running configuration of the JBoss AS in standalone mode.

* Create a management user by running `<jboss-install>/bin/add0user.sh`. 

## Deploy the VDB

* Create a table in local mysql database to query
```sql
CREATE TABLE `Employee` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`id`)
);

INSERT INTO `Employee` VALUES (1,'Jessie'),(2,'Walter'),(3,'Skyler');
``` 

* Deploy the [my-vdb.xml](my-vdb.xml) file using following command. (Provide the username and password of the user created above)
```bash
./<jboss-install>/bin/jboss-cli.sh -c controller=localhost --user=admin --password=admin --command="deploy my-vdb.xml"
```

* Open a browser and go to http://localhost:8080/sample_1/View/get_employees. It will return following json

```json
{
  "data": [
    {
      "id": 1,
      "name": "Jessie"
    },
    {
      "id": 2,
      "name": "Walter"
    },
        {
      "id": 3,
      "name": "Skyler"
    }
  ]
}
```


## References
* [Mysql driver installation](https://medium.com/@hasnat.saeed/install-and-configure-mysql-jdbc-driver-on-jboss-wildfly-e751a3be60d3)
* [Setup teiid](https://docs.jboss.org/author/display/TEIID/Installation+Guide)
* [VDB deployments](https://docs.jboss.org/author/display/TEIID/Deploying+VDBs)