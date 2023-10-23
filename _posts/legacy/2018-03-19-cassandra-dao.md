---
layout: post
title: "Cassandra DAO Example and Unit Test with CassandraUnit"
date: 2018-03-19
categories: [Programming]
tags: [java, cassandra]
---

## What is Cassandra?

Apache Cassandra is a highly-available NoSQL database that has properties of linear scalability and fault tolerance.

**Features**:
* Since Cassandra is a distributed system by nature, it addresses the [CAP](https://en.wikipedia.org/wiki/CAP_theorem) issue by introducing the concept of "Tuning Consistency", which means we can tune the consistency level to anywhere from Eventual Consistency to Strong Consistency.
* Like other NoSQL database, Cassandra needs partitioning key and has loose schema.

The official distribution is Datastax.

Useful resources:
* [Architecture guide](https://docs.datastax.com/en/dse/5.1/dse-arch/datastax_enterprise/dbArch/archTOC.html)
* [How is data read?](https://docs.datastax.com/en/dse/5.1/dse-arch/datastax_enterprise/dbInternals/dbIntAboutReads.html)
* [How is data update? (Why Cassandra write can be faster than read)](https://docs.datastax.com/en/cassandra/3.0/cassandra/dml/dmlWriteUpdate.html)
* [How is data maintained? (The concept of Compaction)](https://docs.datastax.com/en/cassandra/3.0/cassandra/dml/dmlHowDataMaintain.html)
* [Tuning consistency level](https://docs.datastax.com/en/dse/5.1/dse-arch/datastax_enterprise/dbInternals/dbIntConfigConsistency.html)

## Cassandra Driver and DAO example

The official driver for Cassandra is the Datastax version. Here we plant a Dropwizard archetype as a starting point. Full example is available on my GitHub: <https://github.com/lxucs/Tryout-CassandraUnit>

Let's first create simple model classes:

```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class UserIdentifier {

    private String city;

    private String name;

}

@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class User {

    private UserIdentifier identifier;

    private String favoriteFood;

    private int favoriteNumber;
}
```

Create DAO interface for simple operations on User.

```java
public interface UserDao {

    Optional<User> getUser(UserIdentifier identifier);

    List<User> getUsers(String city);

    void updateUser(User user);

}
```

Implement DAO interface using Cassandra driver. The usage of datastax driver is pretty much like JDBC.

```java
public class CassandraUserDaoImpl implements UserDao {

    private final Session session;

    private final PreparedStatement st_getUser;
    private final PreparedStatement st_getUsers;
    private final PreparedStatement st_updateUser;

    public CassandraUserDaoImpl(@NonNull Session session) {
        this.session = session;

        st_getUser = session.prepare("SELECT city, name, favoritefood, favoritenumber " +
                "FROM tryout_cassandraunit.user WHERE city = ? AND name = ?;"
        );

        st_getUsers = session.prepare("SELECT city, name, favoritefood, favoritenumber " +
                "FROM tryout_cassandraunit.user WHERE city = ?;"
        );

        st_updateUser = session.prepare("UPDATE tryout_cassandraunit.user " +
                "SET favoritefood = ?, favoritenumber = ? WHERE city = ? AND name = ?;"
        );
    }

    @Override
    public Optional<User> getUser(@NonNull UserIdentifier identifier) {
        ResultSet rs = session.execute(st_getUser.bind(
                identifier.getCity(),
                identifier.getName()
        ));

        return Optional.ofNullable(mapToUser(rs.one()));
    }

    @Override
    public List<User> getUsers(@NonNull String city) {
        ResultSet rs = session.execute(st_getUsers.bind(city));

        return rs.all().stream().map(r -> mapToUser(r)).collect(Collectors.toList());
    }

    @Override
    public void updateUser(@NonNull User user) {
        session.execute(st_updateUser.bind(
                user.getFavoriteFood(),
                user.getFavoriteNumber(),
                user.getIdentifier().getCity(),
                user.getIdentifier().getName()
        ));
    }

    private User mapToUser(Row r) {
        if(r == null) return null;

        return User.builder()
                .identifier(UserIdentifier.builder()
                        .city(r.getString("city"))
                        .name(r.getString("name"))
                        .build())
                .favoriteFood(r.getString("favoritefood"))
                .favoriteNumber(r.getInt("favoritenumber"))
                .build();
    }
}
```

## Unit Test Using CassandraUnit (Embedded Cassandra Cluster)

To write unit tests for DAO, we either mock the external dependencies (in this case, the Cassandra), or we embed the external dependencies into our application. There is a handy project called [CassandraUnit](https://github.com/jsevellec/cassandra-unit), which can spin up a local Cassandra in JVM, and enables us to unit-test the DAO without mocking anything.

CassandraUnit GitHub and Wiki (noted that some wiki pages are obsolete): <https://github.com/jsevellec/cassandra-unit>

To use the CassandraUnit, we can first take a look at the class [EmbeddedCassandraServerHelper](https://github.com/jsevellec/cassandra-unit/blob/master/cassandra-unit/src/main/java/org/cassandraunit/utils/EmbeddedCassandraServerHelper.java), which is a utility class that can create or cleanup an embedded Cassandra cluster.

In addition, CassandraUnit also has classes that are specifically for JUnit and allows us to load data into the embedded Cassandra. Here we are using the [CassandraCQLUnit](https://github.com/jsevellec/cassandra-unit/blob/master/cassandra-unit/src/main/java/org/cassandraunit/CassandraCQLUnit.java), which enables us to load data by providing a CQL file.

Let's first write the CQL file. Here we create the keyspace, table, and load sample data into the table. We make the city field as partitioning key.

```sql
CREATE KEYSPACE tryout_cassandraunit with replication = {'class':'SimpleStrategy','replication_factor':1};

CREATE TABLE tryout_cassandraunit.user (
city text,
name text,
favoritefood text,
favoritenumber int,
PRIMARY KEY ((city), name)
);

INSERT INTO tryout_cassandraunit.user (city,name,favoritefood,favoritenumber) VALUES ('nyc','user1','kfc',3);
INSERT INTO tryout_cassandraunit.user (city,name,favoritefood,favoritenumber) VALUES ('nyc','user2','popeyes',7);
```

Now let's spin up the local Cassandra and load the data from the CQL file. By using `CassandraCQLUnit`, it can just be one line.

```java
@Rule
public CassandraCQLUnit cassandraCQLUnit = new CassandraCQLUnit(new ClassPathCQLDataSet("LoadDataCassandra.cql", false, false));
```

Once we create the `CassandraCQLUnit`, we can then ask for a Cassandra Session from it, and provide it for our DAO dependency. Therefore, no mocking is needed at all.

Notice that here we handle the keyspace creation and deletion manually. We can also just let CassandraUnit handle it by specifying keyspace name in the `ClassPathCQLDataSet` constructor, so that the below method `cleanUp()` is not necessary.

Here is the full example of the unit test.

```java
public class CassandraUserDaoImplTest {

    public static User user1;
    public static User user2;

    @BeforeClass
    public static void setupUser() {
        user1 = User.builder()
                .identifier(UserIdentifier.builder()
                        .city("nyc")
                        .name("user1")
                        .build())
                .favoriteFood("kfc")
                .favoriteNumber(3)
                .build();

        user2 = User.builder()
                .identifier(UserIdentifier.builder()
                        .city("nyc")
                        .name("user2")
                        .build())
                .favoriteFood("popeyes")
                .favoriteNumber(7)
                .build();
    }

    @Rule
    public CassandraCQLUnit cassandraCQLUnit = new CassandraCQLUnit(new ClassPathCQLDataSet("LoadDataCassandra.cql", false, false));

    /**
     * Clean up keyspace and table after each test, because cql will be loaded before each test.
     */
    @After
    public void cleanUp() {
        EmbeddedCassandraServerHelper.cleanEmbeddedCassandra();
    }

    @Test
    public void testGetUser() {
        Session session = cassandraCQLUnit.getSession();
        UserDao dao = new CassandraUserDaoImpl(session);


        Optional<User> storedUser1 = dao.getUser(UserIdentifier.builder()
                .city("nyc")
                .name("user1")
                .build());

        Assert.assertTrue(storedUser1.isPresent());
        Assert.assertEquals(user1, storedUser1.get());


        Optional<User> storedUser2 = dao.getUser(UserIdentifier.builder()
                .city("nyc")
                .name("user2")
                .build());

        Assert.assertTrue(storedUser2.isPresent());
        Assert.assertEquals(user2, storedUser2.get());
    }

    @Test
    public void testGetUsers() {
        Session session = cassandraCQLUnit.getSession();
        UserDao dao = new CassandraUserDaoImpl(session);

        List<User> users = dao.getUsers("nyc");

        Assert.assertEquals(2, users.size());
        Assert.assertEquals(user1, users.get(0));
        Assert.assertEquals(user2, users.get(1));
    }

    @Test
    public void testUpdateRoundTrip() {
        Session session = cassandraCQLUnit.getSession();
        UserDao dao = new CassandraUserDaoImpl(session);

        User user3 = User.builder()
                .identifier(UserIdentifier.builder()
                        .city("sfo")
                        .name("user3")
                        .build())
                .favoriteFood("subway")
                .favoriteNumber(12)
                .build();

        dao.updateUser(user3);

        Optional<User> storedUser = dao.getUser(UserIdentifier.builder()
                .city("sfo")
                .name("user3")
                .build()
        );

        Assert.assertTrue(storedUser.isPresent());
        Assert.assertEquals(user3, storedUser.get());
    }

}
```

Now let's run `mvn clean verify`. All tests should pass, and application should be built successfully.

![cassandra](/assets/img/legacy/cassandra.png)
