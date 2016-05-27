---
layout: post
title: Custom Junit assertions
categories: [java, development]
tags: [java, junit, hamcrest, assertj]
---

For some reasons it is not possible to assert several aspects of one object within one assert with [JUnit](https://github.com/junit-team/junit/wiki) or even with [Hamcrest](https://code.google.com/p/hamcrest/wiki/Tutorial). I know that a test case should only test one thing, which also can interpreted as one assert, but sometimes it is better to test more aspects of an object instead of duplicate code. e.g.  testing a user with name and gender variables, it is not possible to test them both so if one fails the other is still executed. In this article i will show some ways to solve this problem in a more or less elegant way. If there are some other (better) solution, please let me know! All code sample are available in [github](https://github.com/ccguyka/own-unit-test-assertions).

# Frameworks

As additional frameworks besides the above mentioned ones, I also used [AssertJ](http://joel-costigliola.github.io/assertj/index.html) and [Fest assert](https://github.com/alexruiz/fest-assert-2.x) because they are very powerful and support the [fluent interface ](http://en.wikipedia.org/wiki/Fluent_interface#Java)which produces readable code.

# Code

The code to be tested is very simple and because it is a simple POJO, I would not test it at all in my projects. But for this investigation it will be good enough.

{% highlight java %}
public class User {

    private final String name;
    private final Gender gender;

    public User(String name, Gender gender) {
        this.name = name;
        this.gender = gender;
    }

    public String getName() {
        return name;
    }

    public Gender getGender() {
        return gender;
    }
}

public enum Gender {
    MALE,
    FEMALE;
}
{% endhighlight %}

# Plain old Junit test

The good old JUnit way with _assertEquals_:

{% highlight java %}
@Test
public void failingJunitTest() {
    User user = failingUser();

    assertEquals(MALE, user.getGender());
    assertEquals(NAME, user.getName());
}

private User failingUser() {
    return new User(FAILING_NAME, FEMALE);
}
{% endhighlight %}

It is quit simple and the basic test everyone can write. But there is still the problem that the second assert is not executed all, when the first one fails. The test will fail (as expected) with following message:


    expected:<MALE> but was:<FEMALE>

So the output is quit informative about the error and it should be clear what's wrong with the code.

# Hamcrest matchers

In my opinion the hamcrest matchers should be preferred to the JUnit assert because it provides a [complete list](https://code.google.com/p/hamcrest/wiki/Tutorial) of matchers that will you need while writing tests. And it is much more readable because you can read the assert like a sentence:


    assertThat(number, is(greaterThan(25)));

## Built-in matcher

But the built-in matchers just improves the readability of the test code and in my opinion also the output of the assert message.

{% highlight java %}
@Test
public void failingHamcrestTest() {
    User user = failingUser();

    assertThat(user.getGender(), is(equalTo(MALE)));
    assertThat(user.getName(), is(equalTo(NAME)));
}
{% endhighlight %}

The output is a little bit better because you can easily compare the actual and the expected value.


    Expected: is <MALE>
         but: was <FEMALE>

## Own matcher

But it is also possible to create your own matcher. It is also possible to write your own hamcrest matchers but you have to write one matcher for each of your asserts. So in my opinion there will be too much boilerplate code with this solution. If you want to know more then have a look at this page: [Create your own matcher](http://www.planetgeek.ch/2012/03/07/create-your-own-matcher/)

# AssertJ/FEST assert

I will combine both frameworks because _AssertJ_ is a fork of _FEST assert_ and both implementations will be the same. Both provides a handy fluent interface which will increase the readability of a test.

{% highlight java %}
@Test
public void failingAssertJTest() {
    User user = failingUser();

    assertThat(user).isMale().hasName(NAME);
}
{% endhighlight %}

The output is a better version of the one _JUnit_ provides, because it tries to highlight the difference between expected and actual value.


    org.junit.ComparisonFailure: expected:<[]MALE> but was:<[FE]MALE>

The assertion class can be created in an easy way so there is no excuse if your boss says it takes too much time to create your own assertions.

{% highlight java %}
public class UserAssertJAssertion extends AbstractAssert<UserAssertJAssertion, User> {

    protected UserAssertJAssertion(User actual) {
        super(actual, UserAssertJAssertion.class);
    }

    public static UserAssertJAssertion assertThat(User actual) {
        return new UserAssertJAssertion(actual);
    }

    public UserAssertJAssertion isMale() {
        isNotNull();

        Assertions.assertThat(actual.getGender()).isEqualTo(MALE);

        return this;
    }

    public UserAssertJAssertion hasName(String name) {
        isNotNull();

        Assertions.assertThat(actual.getName()).isEqualTo(name);

        return this;
    }
}
{% endhighlight %}
The examples on the _AssertJ_ website use a different method to do an assert.


    // check condition
    if (!actual.getName().equals(name)) {
       failWithMessage("Expected character's name to be <%s> but was <%s>", name, actual.getName());
    }

I don't know which version is better but that's a kind of your preferences.

# SoftAssertions

As you have seen, all my example will fail because the gender **and** the name are wrong but only the gender seems to be failing. _SoftAssertions_ is a way to solve this problem by collecting all assertions until finished which is done by the _assertAll()_ call.

{% highlight java %}
@Test
public void failingJunitSoftAssertionsTest() {
    User user = failingUser();

    SoftAssertions soft = new SoftAssertions();
    soft.assertThat(user.getGender()).isEqualTo(MALE);
    soft.assertThat(user.getName()).isEqualTo(NAME);
    soft.assertAll();
}
{% endhighlight %}

The output now shows both failures:


    org.assertj.core.api.SoftAssertionError:
    The following 2 assertions failed:
    1) expected:<[]MALE> but was:<[FE]MALE>
    2) expected:<"J[ohn D]oe"> but was:<"J[ane R]oe">

There is one downside of this solution. The first usage of the _SoftAssertions_ class will take some time, on my machine ~300ms instead of 5-50ms.

## Combining fluent interface with soft assertions

The above soft assertion example looks like a normal _JUnit_ test with one additional line. So even this can be improved by creating a _AssertJ_ assertion class.

{% highlight java %}
public class UserAssertJSoftAssertion extends
        AbstractAssert<UserAssertJSoftAssertion, User> {

    private SoftAssertions softAssert;

    protected UserAssertJSoftAssertion(User actual) {
        super(actual, UserAssertJSoftAssertion.class);

        softAssert = new SoftAssertions();
    }

    public static UserAssertJSoftAssertion assertThat(User actual) {
        return new UserAssertJSoftAssertion(actual);
    }

    public UserAssertJSoftAssertion isMale() {
        isNotNull();

        softAssert.assertThat(actual.getGender()).isEqualTo(MALE);

        return this;
    }

    public UserAssertJSoftAssertion hasName(String name) {
        isNotNull();

        softAssert.assertThat(actual.getName()).isEqualTo(name);

        return this;
    }

    public void assertAll() {
        softAssert.assertAll();
    }
}
{% endhighlight %}

Where the actual test method looks like this.

{% highlight java %}
@Test
public void failingAssertJSoftAssertTest() {
    User user = failingUser();

    assertThat(user).isMale().hasName(NAME).assertAll();
}
{% endhighlight %}

# Conclusion

In my opinion, all mentioned frameworks are useful and worth to have a look on it. If it possible try to use all of them because even writing custom assertions is no rocket science and only one class away (in most cases you still have the assertions in your test class so you just need to copy them).

I have seen a presentation about unit testing where following unit test categories are shown:

Basic: JUnit assertion

Advanced: Hamcrest matchers

Expert: Custom assertions

I would put the fluent interface to advanced category.
