---
layout: post
title: Top 10 time consuming JUnit tests
---

In my company we have a some tests that take ages. To discover the most time consuming unit tests I decided to write a small class that prints the top 10 time consuming test methods to maven output.
This is more or less the exercise in chapter 8.9.3 of book [Practical Unit Testing with Junit and Mockito](http://practicalunittesting.com/).

# Implementation

The JUnit `RunListener` is the class of choice to get it done. It provides methods like

- testRunStarted
- testRunFinished
- testStarted
- testFinished

I skip all methods for ignored and failure tests because our test suite is stable.

```java
public class TopTenExecutionTimes extends RunListener {

    private final Map<String, Execution> executionTimes = new HashMap<>();

    @Override
    public void testStarted(Description description) throws Exception {
        executionTimes.put(getKeyFor(description), new Execution());
    }

    @Override
    public void testFinished(Description description) throws Exception {
        Execution executionTime = executionTimes.get(getKeyFor(description));
        if (executionTime != null) {
            executionTime.finish();
        }
    }

    @Override
    public void testRunFinished(Result result) throws Exception {
        List<Map.Entry<String, Execution>> top10 = getTopTen();

        print(top10);
    }

    private void print(List<Map.Entry<String, Execution>> top10) {
        System.out.println();
        System.out.println("Top 10");
        for (Entry<String, Execution> entry : top10) {
            System.out.println(getDurationText(entry.getValue().getDuration()) + "    " + entry.getKey());
        }
    }

    private List<Map.Entry<String, Execution>> getTopTen() {
        List<Map.Entry<String, Execution>> entries = new ArrayList<>(executionTimes.entrySet());
        Collections.sort(entries, (o1, o2) -> o2.getValue().getDuration().compareTo(o1.getValue().getDuration()));
        return entries.subList(0, Math.min(entries.size(), 10));
    }

    private String getKeyFor(Description description) {
        return description.getTestClass().getName() + "." + description.getMethodName();
    }

    private String getDurationText(Long duration) {
        long minutes = TimeUnit.MILLISECONDS.toMinutes(duration);
        long seconds = TimeUnit.MILLISECONDS.toSeconds(duration) % 60;
        long millis = TimeUnit.MILLISECONDS.toMillis(duration) % 1000;

        return String.format("%02d:%02d:%03d", minutes, seconds, millis);
    }

    private class Execution {
        Long start;
        Long end;

        private Execution() {
            start = System.currentTimeMillis();
        }

        void finish() {
            end = System.currentTimeMillis();
        }

        Long getDuration() {
            return end - start;
        }
    }
}
```

# Integration into build

Because we use maven as build tool, I integrated the `TopTenExecutionTimes` class into `maven-surefire-plugin` plugin as a listener.

```xml
<build>
  <plugins>
    ...
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-surefire-plugin</artifactId>
      <version>2.19.1</version>
      <configuration>
        <properties>
          <property>
            <name>listener</name>
            <value>org.ccguyka.testwatcher.TopTenExecutionTimes</value>
          </property>
        </properties>
      </configuration>
    </plugin>
  </plugins>
</build>
```

# Output

```
-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running org.ccguyka.own_unit_test_assertions.UserAssertionTest
Tests run: 10, Failures: 0, Errors: 0, Skipped: 7, Time elapsed: 2.793 sec - in org.ccguyka.own_unit_test_assertions.UserAssertionTest

Top 10
00:01:501  org.ccguyka.own_unit_test_assertions.UserAssertionTest.goodJunitTest
00:01:249  org.ccguyka.own_unit_test_assertions.UserAssertionTest.goodAssertJTest
00:00:013  org.ccguyka.own_unit_test_assertions.UserAssertionTest.goodFestAssertionTest

Results :

Tests run: 10, Failures: 0, Errors: 0, Skipped: 7
```

# Edit (2017/01/07)

To have the code persisted I put the code to my [github](https://github.com/ccguyka/own-unit-test-assertions/blob/master/src/main/java/org/ccguyka/testwatcher/TopTenExecutionTimes.java) project.
