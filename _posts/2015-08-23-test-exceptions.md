---
layout: post
title: Test exceptions
categories: [java, development]
tags: [java, junit, catch-exception]
---

Testing functionality that throws exceptions is quite easy using the expected annotation of [JUnit](https://github.com/junit-team/junit/wiki). But there are also other possibilities to test exceptions:

  * try/catch + fail()
  * expected exception annotation
  * ExpectedException rule
  * [catch-exception](https://github.com/Codearte/catch-exception)

# try/catch + fail()

Catch the thrown exception and call fail() if it is not called is possible with JUnit but shouldn't be used at all. Prefer at least the expected exception solution.

{% highlight java %}
@Test
public void check_thow_exception_if_value_is_null() {
    try {
        testee.methodThatThrowsException(null);

        fail("Should have thrown IllegalArgumentException");
    } catch (IllegalArgumentException e) {
        assertThat(e.getMessage(), is(equalTo("parameter is null")));
    }
}
{% endhighlight %}

# expected exception annotation

The expected exception annotation is the most common version to verify a method throws an exception. Just add the excpetion that is thown to the @Test annotation and violà the exception is catched and the test successful

{% highlight java %}
@Test(expected = IllegalArgumentException.class)
public void check_thow_exception_if_value_is_null() {
    testee.methodThatThrowsException(null);
}
{% endhighlight %}

The drawback of this solution is that the expected result is in front of the actual test. So instead the normal GIVEN/WHEN/THEN you read it like this what will be confusing: THEN/GIVEN/WHEN. And it is not possible to check more than the type of the thrown exeption.

# ExpectedException rule

Another interessting solution provided by JUnit is the ExpectedException rule. The exception to be thrown can be specified by java code instead of using the annotation.

{% highlight java %}
@Rule
public ExpectedException thrown = ExpectedException.none();

@Test
public void check_thow_exception_if_value_is_null() {
    thrown.expect(IllegalArgumentException.class);
    thrown.expectMessage("parameter is null");

    testee.methodThatThrowsException(null);
}
{% endhighlight %}

 It provides more flexibility of checking the expected exception but also have the same problem as the expected exception annotation: The expected result is before the actual test.

# catch-excption

The [catch-exception](https://github.com/Codearte/catch-exception) framework provides a solution that fulfill all needs.

  * Checking the exception after test call
  * Using hamcrest matchers for better readability

{% highlight java %}
@Test
public void check_thow_exception_if_value_is_null() {
    catchException(testee).methodThatThrowsException(null);

    assertThat(caughtException(),
        allOf(
            instanceOf(IllegalArgumentException.class),
            hasMessage("parameter is null")));
}
{% endhighlight %}

It also provides a BDD like interface (when() instead of catchException()) to increase readability.

{% highlight java %}
when(testee).methodThatThrowsException(null);
// vs
catchException(testee).methodThatThrowsException(null);
{% endhighlight %}

# Conclusion

The [catch-exception](https://github.com/Codearte/catch-exception) framework provides a nice and handy solution to test exceptions. It is also possible to use hamcrest matchers and the assertions is after the method call that throws the exception.
