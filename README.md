# Spring Boot based sample backend app running GAE Flex environment using Google SqlCloud

This sample shows how to run a [Spring Boot][spring-boot] application on [Google
Cloud Platform][cloud-java]. It uses the [Google App Engine flexible
environment][App Engine-flexible].

[App Engine-flexible]: https://cloud.google.com/appengine/docs/flexible/
[cloud-java]: https://cloud.google.com/java/
[spring-boot]: http://projects.spring.io/spring-boot/


## Before you begin

This sample assumes you have [Java 8][java8] installed.

[java8]: http://www.oracle.com/technetwork/java/javase/downloads/

### Download Maven

These samples use the [Apache Maven][maven] build system. Before getting
started, be sure to [download][maven-download] and [install][maven-install] wiht maven version 3.x or later.
When you use Maven as described here, it will automatically download the needed
client libraries.

[maven]: https://maven.apache.org
[maven-download]: https://maven.apache.org/download.cgi
[maven-install]: https://maven.apache.org/install.html

### Create a Project in the Google Cloud Platform Console

If you haven't already created a project, create one now. Projects enable you to
manage all Google Cloud Platform resources for your app, including deployment,
access control, billing, and services.

1. Open the [Cloud Platform Console][cloud-console].
1. In the drop-down menu at the top, select **Create a project**.
1. Give your project a name.
1. Make a note of the project ID, which might be different from the project
   name. The project ID is used in commands and in configurations.

[cloud-console]: https://console.cloud.google.com/

### Enable billing for your project.

If you haven't already enabled billing for your project, [enable
billing][enable-billing] now.  Enabling billing allows the application to
consume billable resources such as running instances and storing data.

[enable-billing]: https://console.cloud.google.com/project/_/settings

### Install the Google Cloud SDK.

If you haven't already installed the Google Cloud SDK, [install and initialize
the Google Cloud SDK][cloud-sdk] now. The SDK contains tools and libraries that
enable you to create and manage resources on Google Cloud Platform.

[cloud-sdk]: https://cloud.google.com/sdk/

### Install the Google App Engine SDK for Java


```
gcloud components update app-engine-java
gcloud components update
```

### Configure the `app.yaml` descriptor

The [`app.yaml`][app-yaml] descriptor is used to describe URL
dispatch and resource requirements.  This example sets
[`manual_scaling`][manual-scaling] to 1 to minimize possible costs.
These settings should be revisited for production use.

[app-yaml]: https://cloud.google.com/appengine/docs/flexible/java/configuring-your-app-with-app-yaml
[manual-scaling]: https://cloud.google.com/appengine/docs/flexible/java/configuring-your-app-with-app-yaml#manual-scaling


## The application use google sql cloud (MySQL)

1. Configure to real Google SQL Cloud : visit this https://cloud.google.com/appengine/docs/flexible/java/using-cloud-sql
1. Add a JDBC library and the Cloud SQL MySQL Socket Factory to your application. For example, if you use Maven, you can add the dependency to the project's pom.xml:
```
<dependency>   <!-- http://dev.mysql.com/doc/connector-j/en/ -->
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>6.0.5</version>
</dependency>
<dependency>
  <groupId>com.google.cloud.sql</groupId>
  <artifactId>mysql-socket-factory-connector-j-6</artifactId>
  <version>1.0.2</version>
</dependency>
```
1. Sample connection to MySQL Google Cloud

update connection with pom.xml
```
<properties>
        <database.name>sample</database.name>
        <sqlcloud.instance>flex-helloworld:asia-northeast1:sm-sample</sqlcloud.instance>
        <database.username>root</database.username>
        <database.password>root123</database.password>
        <database.url>jdbc:mysql://google/${database.name}?cloudSqlInstance=${sqlcloud.instance}&amp;socketFactory=com.google.cloud.sql.mysql.SocketFactory</database.url>
      </properties>
      
```

1. Connection to localhost MySQL 

update connection with pom.xml
      
```
<!-- mysql connection localhost -->
    <database.driver>com.mysql.jdbc.Driver</database.driver> 
    <database.url>jdbc:mysql://localhost:3306/local_test?autoReconnect=true&amp;serverTimezone=UTC</database.url>
    <database.username>root</database.username>
    <database.password>root</database.password>
    
```
Datasource : MainApplication.java
```
 @Bean
  public DataSource dataSource() {
      final String propsFile = "db.properties";
      final Properties props = new Properties();
      try {
          props.load(Thread.currentThread().getContextClassLoader().getResource(propsFile).openStream());
          return BasicDataSourceFactory.createDataSource(props);
      } catch (Exception e) {
          System.out.println("Error :" + e.getMessage());
      }
      return null;
  }
 ```
 Other config : db.properties
 
 ```
  //src/main/resources/db.properties
  #driverClassName=${database.driver}
  url=${database.url}
  username=${database.username}
  password=${database.password}

  
  //CaegoryDaoImp.java
  
@Repository("categoryDaoImpl")
public class CategoryDaoImpl implements CategoryDao {
    private static final Logger LOG = LoggerFactory.getLogger(CategoryDaoImpl.class);
    
    private JdbcTemplate jdbcTemplate;
    
    @Autowired
    public CategoryDaoImpl(DataSource dataSource) {
        //jdbcTemplate = new JdbcTemplate(DBCP2DataSourceUtils.getDataSource());
        jdbcTemplate = new JdbcTemplate(dataSource);
    }
    ...
    }
```

## Create MySQL localhost database 

1. import schema.sql

```
mysql -uroot -proot < schema.sql;

```

## Run the application locally

1. Set the correct Cloud SDK project via `gcloud config set project
   YOUR_PROJECT` to the ID of your application.
1. Run `mvn spring-boot:run`
1. Visit http://localhost:8080
1. Visit http://localhost:8080/api/categories/v1/all
1. Visit http://localhost:8080/api/categories/v1/{id}

## Deploy to App Engine flexible environment

1. `mvn appengine:deploy -Ptest`
1. Visit `http://YOUR_PROJECT.appspot.com`.

Note that deployment to the App Engine flexible environment requires the new
[`com.google.cloud.tools:appengine-maven-plugin` plugin][new-maven-plugin].

[new-maven-plugin]: https://cloud.google.com/appengine/docs/flexible/java/using-maven

Java is a registered trademark of Oracle Corporation and/or its affiliates.

