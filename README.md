/actuator - actuator endpoints
# Spring boot admin
/api/applications - spring boot admin application endpoints
/api/applications/<appId> - spring boot admin endpoints by application id
/api/notifications
/api/journal
# Swagger
/swagger-ui.html - swagger UI conviguration
/swagger-resources - version etc.
/swagger-resources/configuration/ui - ui configuration for swagger
/swagger-resources/configuration/security - security configuration for swagger
/v2/api-docs - swagger api endpoint

### UI Endpoints:

- Vaadin UI `localhost:9000/`

### API endpoints

- REST/HATEOAS API `localhost:9000/api`


### Swagger endpoints

- See http://www.baeldung.com/swagger-2-documentation-for-spring-rest-api
- All Swagger Resources(groups) `http://localhost:9000/swagger-resources`
- Swagger UI endpoint: `http://localhost:9000/swagger-ui.html`
- Swagger docs endpoint: `http://localhost:9000/v2/api-docs`

### Actuator endpoints

- See https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-monitoring.html

- Provides a hypermedia-based “discovery page” for the other endpoints. Requires Spring HATEOAS to be on the classpath. Auth: true `http://localhost:9001/actuator`

- Exposes audit events information for the current application. Auth: true  `http://localhost:9001/auditevents`

- Displays an auto-configuration report showing all auto-configuration candidates and the reason why they ‘were’ or ‘were not’ applied. Auth: true `http://localhost:9001/autoconfig`

- Displays a complete list of all the Spring beans in your application. Auth: true `http://localhost:9001/beans`

- Displays a collated list of all @ConfigurationProperties. Auth: true `http://localhost:9001/configprops`

- Performs a thread dump. Auth: true `http://localhost:9001/dump`

- Exposes properties from Spring’s ConfigurableEnvironment. Auth: true `http://localhost:9001/env`

- Shows any Flyway database migrations that have been applied. Auth: true `http://localhost:9001/flyway`

- Shows application health information (when the application is secure, a simple ‘status’ when accessed over an unauthenticated connection or full message details when authenticated). Auth: false `http://localhost:9001/health`

- Displays arbitrary application info. Auth: faslse `http://localhost:9001/info`

- Shows and modifies the configuration of loggers in the application. Auth: true `http://localhost:9001/loggers`

- Shows any Liquibase database migrations that have been applied. Auth: true `http://localhost:9001/liquibase`

- Shows ‘metrics’ information for the current application. Auth: true `http://localhost:9001/metrics`

- Displays a collated list of all @RequestMapping paths. Auth: true `http://localhost:9001/mappings`

- Allows the application to be gracefully shutdown (not enabled by default). Auth: true `http://localhost:9001/shutdown`

- Displays trace information (by default the last 100 HTTP requests). Auth: true `http://localhost:9001/trace`

### Admin console endpoints

- See: http://codecentric.github.io/spring-boot-admin/1.5.4/#getting-started
- Admin console `http://localhost:9000/admin`