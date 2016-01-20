# dropwizard-jwt-cookie-authentication
Dropwizard bundle managing authentication through JWT cookies.

Saving session information in JWT cookies allows your server to remain stateless, which comes with a lot of advantages regarding scalability and memory usage.

## Enabling the bundle

### Add the dropwizard-jwt-cookie-authentication dpendency

Add the dropwizard-jwt-cookie-authentication library as a dependency to your `pom.xml` file:

```xml
<dependency>
    <groupId>org.dhatim</groupId>
    <artifactId>dropwizard-jwt-cookie-authentication</artifactId>
    <version>1.0.0</version>
</dependency>
  ```

### Edit you app's Dropwizard YAML config file (not required, default values are shown below)

```yml
jwtCookieAuth:
  secretSeed: null
  httpsOnlyCookie: false
  sessionExpiryVolatile: PT30m
  sessionExpiryPersistent: P7d
```

### Add the 'JwtCookieAuthConfiguration' to your application configuration class:
```java
@Valid
@NotNull
private JwtCookieAuthConfiguration jwtCookieAuth = new JwtCookieAuthConfiguration();

public JwtCookieAuthConfiguration getJwtCookieAuth() {
  return jwtCookieAuth;
}
```

### Add the bundle to the dropwizard application

```java
public void initialize(Bootstrap<MyApplicationConfiguration> bootstrap) {
  bootstrap.addBundle(new JwtCookieAuthBundle<>(MyApplicationConfiguration::getJwtCookieAuth);
}
```

## Using the bundle

The authentication result is an instance of `Subject` which must be initially put in the security context using `requestContext.setSecurityContext`
```java
requestContext.setSecurityContext(
  new JwtCookieSecurityContext(
    new Subject(subjectName),
    requestContext.getSecurityContext().isSecure()
  )
);
```

Once a subject has been set, it can be retrieved using the `@Auth` annotation in method signatures.

Each time an API endpoint is called, a fresh cookie JWT is issued to reset the session TTL. You can use the `@DontRefreshSession` on methods where this behavior is unwanted.

To specify a max age in the cookie (aka "remember me"), use `Subject.setLongTermToken(true)`.

Subject roles can be specified via the `Subject.setRoles(...)` method. You can then define fine grained access control using annotations such as `@RolesAllowed` or `@PermitAll`.

Additional custom data can be stored in the Subject using `Subject.getClaims().put(key, value)`.

## Sample application resource
```java
@POST
@Consumes(MediaType.APPLICATION_JSON)
@Produces(MediaType.APPLICATION_JSON)
public Subject setSubject(@Context ContainerRequestContext requestContext, String subjectName){
    Subject subject = new Subject(subjectName);
    requestContext.setSecurityContext(
      new JwtCookieSecurityContext(
        new Subject(subjectName),
        requestContext.getSecurityContext().isSecure()
      )
    );
    return subject;
}

@GET
@Produces(MediaType.APPLICATION_JSON)
public Subject getSubject(@Auth Subject subject){
    return subject;
}

@GET
@Path("idempotent")
@Produces(MediaType.APPLICATION_JSON)
@DontRefreshSession
public Subject getSubjectWithoutRefreshingSession(@Auth Subject subject){
    return subject;
}

@GET
@Path("restricted")
@RolesAllowed("admin")
public String getRestrisctedResource(){
    return "SuperSecretStuff";
}
```

## JWT Signing Key

By default, the signing key is randomly generated on application startup. It means that users will have to re-authenticate after each reboot.

To avoid this, you can specify a `secretSeed` in the configuration. This seed will be used to generate the signing key, which will therefore be the same at each application startup.

Alternatively you can specify your own key factory:
```java
bootstrap.addBundle(new JwtCookieAuthBundle<>(MyApplicationConfiguration::getJwtCookieAuth).setKeyFactory((configuration, environment) -> {/*return your own key*/}));
```

## Javadoc

It's [here](http://dhatim.github.io/dropwizard-jwt-cookie-authentication).