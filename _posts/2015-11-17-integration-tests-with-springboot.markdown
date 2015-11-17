---
layout: post
title:  "Integration tests in Spring. How to do it?"
date:   2015-11-17 18:52:45
categories: spring
---
Recently at Espeo, we wanted to write integration tests, and a connection with the database was required in almost all  aspects of business logic. 
Writing only unit tests without a DB connection didn't make much sense - we were only able to check if the API returns a proper error message in cases like no records or unsuccessful authentication. Therefore, it was necessary to test in using the database. 

It's important to test the DB connection without changing the structure of the production database – but how can this be done? The anwser is quite simple - use two databases – the production one and the test one.


Our project is written in Java, using the Spring 4 platform. 
First, I created a test database using dump from the original one. 
I prepared scripts to create, recreate and drop this database as needed, which were added to Makefile.

Now, how can you force the application to use the regular database in all cases except test, where it should use the test database? You just have to swap the database.properties file.
My original database.properties looked like this:

{% highlight java %}
datasource.driver-class-name=org.postgresql.Driver
datasource.url=jdbc:postgresql://127.0.0.1:5432/my_database
datasource.username=my_username
datasource.password=my_pass
{% endhighlight %}

I wanted to change it to:

{% highlight java %}
datasource.driver-class-name=org.postgresql.Driver
datasource.url=jdbc:postgresql://127.0.0.1:5432/test_db
datasource.username=test_username
datasource.password=test_pass
{% endhighlight %}

We can use a different property (not only for the database, but for everything else as well) using Spring annotation from spring-test dependency.
The test class should have 3 annotations:

@RunWith(SpringJUnit4ClassRunner.class) - Indicates that the class should use Spring's JUnit facilities

@ContextConfiguration (Application.class): Indicates ApplicationContext.

My Application.class looks like this:

{% highlight java %}
@Configuration
@PropertySources({
    @PropertySource("classpath:database.properties")
    @PropertySource("classpath:application.properties”)
})
@ComponentScan
@EnableAutoConfiguration
public class Application extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(
        return application.sources(Application.class);
    }

}


{% endhighlight %}

To override original properties, you just should use @TestPropertySource annotation to the set of PropertySources for an integration test. 

The test property sources have a higher priority than those from the system environment, Java system and property sources added  declaratively or programmatically using @PropertySource annotation. Overriding single classes to use different property sources works perfectly.

So, the entire test looked more less like this:

{% highlight java %}
@@RunWith(SpringJUnit4ClassRunner.class)
@TestPropertySource("classpath:database-test.properties")
@ContextConfiguration(classes = Application.class)
public class SomethingIntegrationTest {

    @Autowired
    private SomeRepo repo;

    @Test
    public void should_return_proper_something() {
        List<SomeModel> allSomethings = repo.getAllSomethings();
        assertThat("Somethings list should return 7 records", allSomethings.size(), is(7));
    }
}
{% endhighlight %}


I added this dependency to pom.xml file:

{% highlight java %}
<dependency>
            <groupId>org.springframework</
            <artifactId>spring-test</
 </dependency>
{% endhighlight %}




