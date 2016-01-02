# String Builder QL Params
[![kr.pe.kwonnam.jspMaven Central](https://maven-badges.herokuapp.com/maven-central/kr.pe.kwonnam.dynamicql/string-builder-ql-params/badge.svg)](https://maven-badges.herokuapp.com/maven-central/kr.pe.kwonnam.dynamicql/string-builder-ql-params)

Java String Builder QL Params helps building SQL/JPQL/HQL with [java.lang.StringBuilder](http://docs.oracle.com/javase/7/docs/api/java/lang/StringBuilder.html).

## Requirements
* Java 6+

## Dependency Configuration
### Gradle
```groovy
dependencies {
    compile 'kr.pe.kwonnam.dynamicql:string-builder-ql-params:0.1'
}
```

### Maven
```xml
<dependency>
    <groupId>kr.pe.kwonnam.dynamicql</groupId>
    <artifactId>string-builder-ql-params</artifactId>
    <version>0.1</version>
    <scope>compile</scope>
</dependency>
```

## DynamicQlParams
Use [DynamicQlParams](https://github.com/kwon37xi/string-builder-ql-params/blob/master/src/main/java/kr/pe/kwonnam/dynamicql/stringbuilderqlparams/DynamicQlParams.java) for building positional parameter based SQL/JPQL/HQL.
If you don't want to depend on this project, just copy&paste the source to your project.

### Creating instances

```java
StringBuilder builder = new StringBuilder();

// instantiating for normal positional parameters(like '?')
DynamicQlParams dqp = new DynamicQlParams();

// instanticating for JQPL/HQL's indexed positional parameters(like '?1', '?2', ...)
DynamicQlParams dqp = new DynamicQlParams(true);
// or
DynamicQlParams dqp = DynamicQlParams.withPositionalIndex();
```

### API for `DynamicQlParams`
* `DynamicQlParams.param(Object param)` : Add a parameter value.
* `DynamicQlParams.inParams(Object[] params or Iterable<?> params)` : Add parameter values for IN condition.
* `DynamicQlParams.getParameters()` : get the final parameter values as `java.util.List`
* `DynamicQlParams.getParameterArray()` : get the final parameter values as an array.
* `DynamicQlParams.bindParameters(PreparedStatement preparedStatement)` : bind parameters to a preparedStatement object.

### Example Usage
You can get the fowlloing source from [DynamicQlParamsQueryBuildTest.java](https://github.com/kwon37xi/string-builder-ql-params/blob/master/src/test/java/kr/pe/kwonnam/dynamicql/stringbuilderqlparams/DynamicQlParamsQueryBuildTest.java).

```java
// Let's assume that you have the following data
User user = new User();
user.setUserId(2015L);
user.setName("DynamicQlParams");
user.setBirthday(new SimpleDateFormat("yyyy/MM/dd").parse("2015/12/17"));
user.setEmail("someone@email.com");

List<String> zipCodes = Arrays.asList("12345", "56789", "58391");

StringBuilder builder = new StringBuilder();
DynamicQlParams dqp = new DynamicQlParams();

builder
    .append("SELECT ")
    .append(StringUtils.join(User.COLUMNS, ", ")).append("\n")
    .append("FROM users as u\n")
    .append("WHERE 1 = 1\n");

if (user.getUserId() != null) {
    builder.append("AND user_id = ").append(dqp.param(user.getUserId())).append("\n");
}
if (StringUtils.isNotEmpty(user.getName())) {
    builder.append("AND name = ").append(dqp.param(user.getName())).append("\n");
}
if (user.getBirthday() != null) {
    builder.append("AND birthday = ").append(dqp.param(user.getBirthday())).append("\n");
}
if (CollectionUtils.isNotEmpty(zipCodes)) {
    builder.append("AND zip_code in (").append(dqp.inParams(zipCodes)).append(")\n");
}
builder.append("LIMIT ").append(dqp.param(10));

log.info("StringBuilder with DynamicQlParams : {}", builder.toString());
log.info("Query Parameters : {}", dqp.getParameters());

PreparedStatement pstmt = connection.prepareStatement(builder.toString());
// you can use this parameters with PreparedStatement or Spring JdbcTemplate or Hibernate session or JPA entityManager, etc..
dqp.bindParameters(preparedStatement);

// execute query...
```

You will get the following query string

```sql
SELECT user_id, name, email, birthday, mobile_phone, home_phone, address, zip_code
FROM users as u
WHERE 1 = 1
AND user_id = ?
AND name = ?
AND birthday = ?
AND zip_code in (?, ?, ?)
LIMIT ?
```

but if you set `withPositionalIndex = true`, the you will get the following query string

```sql
SELECT user_id, name, email, birthday, mobile_phone, home_phone, address, zip_code
FROM users as u
WHERE 1 = 1
AND user_id = ?1
AND name = ?2
AND birthday = ?3
AND zip_code in (?4, ?5, ?6)
LIMIT ?7
```

and parameters

```
[2015, DynamicQlParams, Thu Dec 17 00:00:00 KST 2015, 12345, 56789, 58391, 10]
```

## DynamicNamedQlParams
Use [DynamicNamedQlParams](https://github.com/kwon37xi/string-builder-ql-params/blob/master/src/main/java/kr/pe/kwonnam/dynamicql/stringbuilderqlparams/DynamicNamedQlParams.java) for building named parameter based SQL/JPQL/HQL.
If you don't want to depend on this project, just copy&paste the source to your project.

JPQL, HQL and Spring's [NamedParameterJdbcTemplate](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/jdbc/core/namedparam/NamedParameterJdbcTemplate.html) supports named parameters.

### API for `DynamicNamedQlParams`
* `DynamicNamedQlParams.param(String paramName, Object param)` : Add a named parameter value.
* `DynamicNamedQlParams.inParams(Object[] params or Iterable<?> params)` : Add named parameter values for IN condition.
* `DynamicNamedQlParams.getParameters()` : get the final parameter values as `java.util.Map`

### Example Usage

You can get the fowlloing source from [DynamicNamedQlParamsQueryBuildTest.java](https://github.com/kwon37xi/string-builder-ql-params/blob/master/src/test/java/kr/pe/kwonnam/dynamicql/stringbuilderqlparams/DynamicNamedQlParamsQueryBuildTest.java).

```java
StringBuilder builder = new StringBuilder();
DynamicNamedQlParams dnqp = new DynamicNamedQlParams();

builder
    .append("SELECT ")
    .append(StringUtils.join(User.COLUMNS, ", ")).append("\n")
    .append("FROM users as u\n")
    .append("WHERE 1 = 1\n");

if (user.getUserId() != null) {
    builder.append(format("AND user_id = %s %n", dqp.param(user.getUserId())));
}
if (StringUtils.isNotEmpty(user.getName())) {
    builder.append(format("AND name = %s %n", dqp.param(user.getName())));
}
if (user.getBirthday() != null) {
    builder.append(format("AND birthday = %s %n", dqp.param(user.getBirthday())));
}
if (CollectionUtils.isNotEmpty(zipCodes)) {
    builder.append(format("AND zip_code in (%s) %n",dqp.inParams(zipCodes)));
}
builder.append("LIMIT ").append(dnqp.param("limit", 10));

log.info("StringBuilder with DynamicNameQlParams : {}", builder.toString());
log.info("Query Parameters : {}", dnqp.getParameters());
```

You will get the following query string

```sql
SELECT user_id, name, email, birthday, mobile_phone, home_phone, address, zip_code
FROM users as u
WHERE 1 = 1
AND user_id = :userId
AND name = :name
AND birthday = :birthday
AND zip_code in (:zipCode0, :zipCode1, :zipCode2)
LIMIT :limit
```

and parameters

```
{limit=10, birthday=Thu Dec 17 00:00:00 KST 2015, name=DynamicQlParams, userId=2015, zipCode1=56789, zipCode0=12345, zipCode2=58391}
```
