---
title: Organising Sequential Code Flow in Java with Easy-Workflow
date: 2020-12-25 16:51:18
category:
- Languages
- Java
tags:
- "#java"
- "#spring"
---

## The Philosophy

_:grey_exclamation: Skip this section if you are not interested in the philosophy of good software engineering, and want to jump straight to the relevant stuff._

Software engineering _(the art)_ is an interesting space in many aspects. One such aspect is the fact that the same tools can be used to solve a myriad of problems. To solve these problems, there can be numerous ways, none being perfect. At the same time, every solution has scope for improvement.

Software engineers' _(the artist)_ task is to take care of all these trade-offs and select the best approach to solve the business use-cases in a reasonable time, along with taking care of the software best practices along the way.

Enough of philosophy. Let's get to the point now.

## The Lazy, Long Step-Based Methods

Most of us may have come across situations where we need to implement algorithms that are sequential in manner. They simply require you to implement step 1, then step 2 and so on.

A naive implementation can simply have a method that handles all the steps. You can already see this method bloating, forget about its future maintenance.

In a relatively more mature implementation, the method call will be broken down into sub-methods, and the code may look something like this:

```java
class SomeAlgorithm {
    public void runAlgorithm() {
        performStep1();
        performStep2();
        performStep3();
        // and so on...
    }
        
    private void performStep1() {
        // ...
    }
    
    private void performStep2() {
        // ...
    }
    
    private void performStep3() {
        // ...
    }
}
```

Okay still better. But _(if you read the philosophy section above)_, no solution is perfect. Do you see where this approach can go wrong?

Let's take an example. Suppose `performStep1()` requires you to gather data from some third party application, and then in the rest of the steps you are actually performing the algorithm. Maybe in one of the last few steps you want to write to the database. See what's happening? This class is now handling so many things apart from the algorithm itself. _Separation of concerns?_ Boom. Lost. 

Hmm. So what can we do about it? Maybe we start thinking in terms of Object Oriented Principles and create different classes for each of these not-so-related tasks. This approach does make sense but only to the extent in which you were dealing with such a scenario once or twice in an application.

What if you expect the implementation of such algorithms at many places inside the codebase? So many classes, each handling different logic; but still kind of similar be a part of one single long-running algorithm. Feels chaotic already?

That's where you need to start considering such problems as long-running tasks, which will not only help in doing away with bloated classes but also managing the algorithms in a streamlined manner. 

This was one such problem I faced recently, and this article is about how I went on looking at these problems as "workflows" and creating a common solution for implementing them. I created a solution which I call the "Easy-Workflow".

## Easy-Workflow - goodbye long methods!

### Why?

Many Java enterprise applications require processing to be executed in a context separate from that of the main system. In many cases, these backend processes perform several tasks, with some tasks dependent upon a previous task's status. With the requirement of interdependent processing tasks, an implementation using a single procedural-style set of method calls usually proves inadequate. Utilizing Spring, a developer can easily separate a backend process into an aggregation of activities. Easy-Workflow helps to streamline this process.

### What?

Easy-Workflow aims to make sequential and/or parallel execution of independent yet related activities in a more organized manner. It lets you define your desired workflow with independent activities. The definition of these activities can be provided by the client as per business needs. It comes with functionality such as contexts and customizable error handlers.

### When to use?

In Easy-Workflow's jargon, a workflow can be defined as a set of activities performed in a predetermined order without user interaction.

This approach, however, is not suggested as a replacement for existing workflow frameworks. For scenarios where more advanced interactions are necessarily based on user input, a standalone open-source or commercial workflow engine is better equipped.

If the workflow tasks at hand are simplistic, then the Easy-Workflow approach makes more sense, especially if Spring is already in use.
 
### Requirements

* Java
* Spring
* Easy-Workflow JAR

### Features

* Interact with Spring using both XML-based Configuration and Annotation-based Configuration
* Pass information between activities using _WorkflowContext_
* Customize _ErrorHandlers_ at a per-activity level as well as a default-workflow level
* Get detailed _WorkflowReport_ once execution of Workflow is over

## Architecture

Easy-Workflow consists of the following components:

### Workflow Engine
A place to kick-start your workflow.

### Workflow
A set of activities performed in a predetermined order without user interaction. The activities can be configured to either run in sequence, or in parallel, or using a hybrid approach.

### Activity
A single unit of work that can be performed independently. This is where the business logic needs to be defined.

### WorkflowContext
A utility to pass around information between different activities of a workflow. It can also be used to provide a seed to the first activity of the workflow.

### WorkflowReport
A unit that gives all details about how the workflow finished and contains any messages/errors to be returned to the caller.

### WorkflowStatus
An enum to represent the status in which the workflow could have ended like completed, failed, stopped, or no operation.

## Usage

### Simple Workflow with XML-based Configuration

Let's take an example of a very simple workflow where three activities run in a sequential manner, and any error encountered while running these activities needs to be handled gracefully. We will explore the implementation with the help of an XML-based configuration. 

![](SimpleWorkflow.png)

#### beans.xml

This is the starting point of the workflow, where the bean definitions, as well as the workflow definition, is provided. In the first few lines, we have the bean definitions of `Activity1`, `Activity2`, `Activity3` along with `SimpleErrorHandler` and `SimpleContext`. Here, activities are custom implementations that will contain the business logic, and `SimpleErrorHandler` and `SimpleContext` are very basic implementations of _ErrorHandler_ and _WorkflowContext_, respectively.

In the latter half of the code, we can see the definition of the workflow. The property `activities` is a list that actually defines all the activities (beans) that will run. They run in the order of the sequence defined in this list. This is the section which ties all the components of a workflow together, hence the mapping of _ErrorHandler_ and _WorkflowContext_ beans also exists here.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="activity1" class="deeheem.coffeestore.examples.simple.Activity1"/>

    <bean id="activity2" class="deeheem.coffeestore.examples.simple.Activity2"/>

    <bean id="activity3" class="deeheem.coffeestore.examples.simple.Activity3"/>

    <bean id="defaultErrorHandler" class="deeheem.coffeestore.examples.simple.SimpleErrorHandler"/>

    <bean id="simpleContext" class="deeheem.coffeestore.examples.simple.SimpleContext" scope="prototype"/>


    <!-- simple workflow  -->
    <bean id="simpleWorkflow" class="org.deeheem.easyworkflow.domain.workflow.DefaultWorkflow">
        <property name="activities">
            <list>
                <ref bean="activity1"/>
                <ref bean="activity2"/>
                <ref bean="activity3"/>
            </list>
        </property>
        <property name="defaultErrorHandler">
            <ref bean="defaultErrorHandler"/>
        </property>
        <property name="workflowContext">
            <ref bean="simpleContext" />
        </property>
    </bean>
</beans>
```

#### Activity1.java

Notice that the activities need to extend `BaseActivity` class which implements the interface `Activity`. `Base Activity` is an abstract implementation of `Activity` designed for re-use by business-specific activities. Both `BaseActivity` and `Activity` are present in the Easy-Workflow JAR itself.

```java
public class Activity1 extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        // business logic goes here
        System.out.println("EXECUTE Activity1");
        SimpleContext simpleContext = (SimpleContext) context;
        simpleContext.put("message1", "Data from Activity1");
    }
}
```

#### Activity2.java

```java
public class Activity2 extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        // business logic goes here
        System.out.println("EXECUTE Activity2");
        SimpleContext simpleContext = (SimpleContext) context;
        simpleContext.put("message2", "Data from Activity2");
    }
}
```

#### Activity3.java

```java
public class Activity3 extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        // business logic goes here
        System.out.println("EXECUTE Activity3");
        SimpleContext simpleContext = (SimpleContext) context;
        for (Map.Entry<String, Object> entry : simpleContext.getEntrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }
        // uncomment the below line to invoke the error handler
        // throw new Exception("let's see the error");
    }
}
```

#### SimpleContext.java

SimpleContext implements the interface `WorkflowContext` provided in the Easy-Workflow JAR. 
* `setSeedData()` method can be used to provide information to the workflow at the time of kickoff, which may be required by other activities during the workflow run.
* `shouldWorkflowStop()` method is used by individual activities to inform the workflow to stop processing further due to some requirement/condition reached in the business logic. Note that this is **not the same as** the ability of the error handler to stop the workflow in case an error is encountered (something we will see in the next section). Instead, this particular function is used for handling business logic conditions rather than error scenarios.
 
```java
public class SimpleContext implements WorkflowContext {
    public static final String SEED_DATA = "seedData";

    private AtomicBoolean stopProcess;
    private final Map<String, Object> context = new ConcurrentHashMap<>(); // to make it thread-safe

    @Override
    public void setSeedData(Object seedObject) {
        this.put(SEED_DATA, seedObject);
    }

    @Override
    public AtomicBoolean shouldWorkflowStop() {
        return stopProcess;
    }

    public void setStopProcess(AtomicBoolean stopProcess) {
        this.stopProcess = stopProcess;
    }

    public void put(String key, Object value) {
        context.put(key, value);
    }

    public Object get(String key) {
        return context.get(key);
    }

    public Set<Map.Entry<String, Object>> getEntrySet() {
        return context.entrySet();
    }
}
```

#### SimpleErrorHandler.java

SimpleErrorHandler implements the interface `ErrorHandler` provided in the Easy-Workflow JAR, which defines a strategy for handling errors/exceptions which may arise during the execution of an activity.  
* `handleError()` handles the given error, possibly rethrowing it as a fatal exception.
* It is up to the developer to decide how to handle the error inside this method.
* This method can also be used to handle errors from composite activities like `ConcurrentActivity` (discussed in upcoming sections).
* If you notice, the return type of this method is `boolean`. This specifies whether the execution of workflow needs to be stopped on encountering an error, and can be customized by the developer implementing the business logic based on the needs.

```java
public class SimpleErrorHandler implements ErrorHandler {
    private String beanName;

    @Override
    public boolean handleError(WorkflowContext context, Exception exception) {
        throw new UnsupportedOperationException();
    }

    public void setBeanName(String beanName) {
        this.beanName = beanName;
    }
}
```

### Complex Workflow with XML-based Configuration

Now let's take a more complicated example consisting of parallel activity runs, in a combination with some activities which run in sequence.

![](ComplexWorkflow.png)

As shown in the diagram above, two activities run in parallel and once both of them have completed their execution, the flow then passes to the other two activities. Notice that the error handlers are different for concurrent and sequential executions.

On a side note, you can also define your own custom error handler on as many of the activities as you like.

#### beans.xml

Similar to the previous example, this is the starting point of the workflow. Let's focus on the special beans here.

`c_activity1` represents Activity 1 which runs two sub-activities concurrently. We define these sub-activities inside the property `parallelActivities` as list, although note that the order inside this list does not matter during execution since the beans in the list `c_pActivity1` and `c_pActivity2` will run in parallel.

Inside the workflow definition, we have defined activities `c_activity1`, `c_activity2` and `c_activity3` to run in sequence, hence completing our objective of running the workflow as per the diagram.

Note that in the case of complex workflows with concurrent activities, we have the option to define the `maxThreadPoolSize` to control how many threads at the maximum do we want to spawn during the workflow run.


```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

    <bean id="c_pActivity1" class="deeheem.coffeestore.examples.complex.PActivity1"/>

    <bean id="c_pActivity2" class="deeheem.coffeestore.examples.complex.PActivity2"/>

    <bean id="c_activity1" class="deeheem.coffeestore.examples.complex.Activity1">
        <property name="parallelActivities">
            <list>
                <ref bean="c_pActivity1"/>
                <ref bean="c_pActivity2"/>
            </list>
        </property>
        <property name="errorHandler">
            <ref bean="c_concurrentErrorHandler"/>
        </property>
    </bean>

    <bean id="c_activity2" class="deeheem.coffeestore.examples.complex.Activity2"/>

    <bean id="c_activity3" class="deeheem.coffeestore.examples.complex.Activity3"/>

    <bean id="c_defaultErrorHandler" class="deeheem.coffeestore.examples.complex.SimpleErrorHandler"/>

    <bean id="c_concurrentErrorHandler" class="deeheem.coffeestore.examples.complex.ConcurrentErrorHandler"/>

    <bean id="c_simpleContext" class="deeheem.coffeestore.examples.complex.SimpleContext" scope="prototype"/>

    <!-- complex workflow  -->
    <bean id="complexWorkflow" class="org.deeheem.easyworkflow.domain.workflow.DefaultWorkflow">
        <property name="activities">
            <list>
                <ref bean="c_activity1"/>
                <ref bean="c_activity2"/>
                <ref bean="c_activity3"/>
            </list>
        </property>
        <property name="defaultErrorHandler">
            <ref bean="c_defaultErrorHandler"/>
        </property>
        <property name="workflowContext">
            <ref bean="c_simpleContext" />
        </property>
        <property name="maxThreadPoolSize" value="4" />
    </bean>
</beans>
```

#### PActivity1.java

```java
public class PActivity1 extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("EXECUTE PActivity1");
        SimpleContext simpleContext = (SimpleContext) context;
        simpleContext.put("message1", "Data from PActivity1");

        // note: uncomment below lines to see how exception is handled in parallel activities
        //        int b=0;
        //        try {
        //            int c = 10 / b;
        //        } catch (Exception e) {
        //            System.out.println("test exception");
        //            throw new Exception("test exception", e);
        //        }
    }
}
```

#### PActivity2.java

```java
public class PActivity2 extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("EXECUTE PActivity2");
        SimpleContext simpleContext = (SimpleContext) context;
        simpleContext.put("message2", "Data from PActivity2");
    }
}
```

#### Activity1.java

Note that `Activity1` needs to extend `ConcurrentActivity` in order to support concurrent execution of the sub-activities.

```java
public class Activity1 extends ConcurrentActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("EXECUTE Activity1");
        SimpleContext simpleContext = (SimpleContext) context;
        simpleContext.put("message1", "Data from Activity1");
    }
}
```

#### Activity2.java

```java
public class Activity2 extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("EXECUTE Activity2");
        SimpleContext simpleContext = (SimpleContext) context;
        simpleContext.put("message2", "Data from Activity2");
    }
}
```

#### Activity3.java

```java
public class Activity3 extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("EXECUTE Activity3");
        SimpleContext simpleContext = (SimpleContext) context;
        for (Map.Entry<String, Object> entry : simpleContext.getEntrySet()) {
            System.out.println(entry.getKey() + ": " + entry.getValue());
            System.out.println(entry.getKey() + ": " + entry.getValue());
        }
        //   throw new Exception("let's see the error");
    }
}
```

#### SimpleContext.java

```java
public class SimpleContext implements WorkflowContext {
    public static final String SEED_DATA = "seedData";

    private AtomicBoolean stopProcess;
    private final Map<String, Object> context = new ConcurrentHashMap<>(); // to make it thread-safe

    @Override
    public void setSeedData(Object seedObject) {
        this.put(SEED_DATA, seedObject);
    }

    @Override
    public AtomicBoolean shouldWorkflowStop() {
        return stopProcess;
    }

    public void setStopProcess(AtomicBoolean stopProcess) {
        this.stopProcess = stopProcess;
    }

    public void put(String key, Object value) {
        context.put(key, value);
    }

    public Object get(String key) {
        return context.get(key);
    }

    public Set<Map.Entry<String, Object>> getEntrySet() {
        return context.entrySet();
    }
}
```

#### SimpleErrorHandler.java

```java
public class SimpleErrorHandler implements ErrorHandler {
    private String beanName;

    @Override
    public boolean handleError(WorkflowContext context, Exception exception) {
        throw new UnsupportedOperationException();
    }

    public void setBeanName(String beanName) {
        this.beanName = beanName;
    }
}
```

### Real-World Workflow with Annotation-based Configuration

_:grey_exclamation: Disclaimer: The example in this section, even though termed as a "Real-World Workflow", is still fictional and may not directly apply to suit similar use-cases._

Imagine you are a developer of an app for a store specializing in different types of coffee. The store curates the best coffee from all over the world, and customers have the ability to subscribe to a monthly newsletter from where they can try out a new type of coffee every month. You, being very particular about the preferences of each customer, want the newsletter to be customised to each customer. _(Before we digress, we are not going the Machine Learning way. This is NOT an article on Machine Learning.)_ 

Now as always, the problem has many nuances and can be solved in multiple ways. Let's explore one such way a typical store might want to solve it: from all the coffee available at the store, enhance the existing data with customer's preferences, manufacturer details, tailored prices, and then save the details for notifying the customers at a relevant point in time.

If we think of the Easy-Workflow approach, we can easily break down the problem in the following way:


![](RealWorkflow.png)

Makes sense? 

Note that whenever you trying to break down the problem into a flowchart like this to make use of Easy-Workflow, make sure that you create a Directed Acyclic Graph (DAG).

#### CoffeeNewsletterConfiguration.java

Let's see how we can create the beans for this application using annotation-based approach.

```java
@Configuration
public class CoffeeNewsletterConfiguration {

    // ---- Error Handlers ----

    @Bean(value = "parallelErrorHandler")
    public ErrorHandler parallelErrorHandler() {
        return new ParallelErrorHandler();
    }

    @Bean(value = "fallbackErrorHandler")
    public ErrorHandler fallbackErrorHandler() {
        return new FallbackErrorHandler();
    }

    // ---- Activities ----

    @Bean(value = "activity_A")
    public BaseActivity coffeeDetailsActivity() {
        return new CoffeeDetailsActivity();
    }

    @Bean(value = "activity_B1")
    public BaseActivity manufacturerDetailsActivity() {
        return new ManufacturerDetailsActivity();
    }
    @Bean(value = "activity_B2")
    public BaseActivity priceCalculatorActivity() {
        return new PriceCalculatorActivity();
    }

    @Bean(value = "activity_B3")
    public BaseActivity customerPreferenceActivity() {
        return new CustomerPreferenceActivity();
    }

    @Bean(value = "activity_B")
    public BaseActivity parallelDataActivity(@Qualifier("activity_B1") BaseActivity activityA,
                                            @Qualifier("activity_B2") BaseActivity activityB,
                                            @Qualifier("activity_B3") BaseActivity activityC,
                                            @Qualifier("parallelErrorHandler") ErrorHandler errorHandler) {
        List<BaseActivity> activities = new ArrayList<>();
        activities.add(activityA);
        activities.add(activityB);
        activities.add(activityC);
        return new ParallelDataActivity(activities, errorHandler);
    }

    @Bean(value = "activity_C")
    public BaseActivity combineCoffeeDetailsActivity() {
        return new CombineCoffeeDetailsActivity();
    }

    @Bean(value = "activity_D")
    public BaseActivity saveActivity() {
        return new SaveActivity();
    }

    // ---- Context ----

    @Bean(value = "workflowContext")
    public WorkflowContext workflowContext() {
        return new ThisWorkflowContext();
    }

    // ---- Workflow ----

    @Bean
    public DefaultWorkflow coffeeNewsletterWorkflow(
            @Qualifier("workflowContext") WorkflowContext workflowContext,
            @Qualifier("fallbackErrorHandler") ErrorHandler errorHandler,
            @Qualifier("activity_A") BaseActivity activityA,
            @Qualifier("activity_B") BaseActivity activityB,
            @Qualifier("activity_C") BaseActivity activityC,
            @Qualifier("activity_D") BaseActivity activityD) {
        List<Activity> activityList = new ArrayList<>();
        activityList.add(activityA);
        activityList.add(activityB);
        activityList.add(activityC);
        activityList.add(activityD);

        DefaultWorkflow defaultWorkflow = new DefaultWorkflow();
        defaultWorkflow.setActivities(activityList);
        defaultWorkflow.setDefaultErrorHandler(errorHandler);
        defaultWorkflow.setWorkflowContext(workflowContext);

        return defaultWorkflow;
    }
}
```

Notice that `ParallelErrorHandler` is being used for handling errors occurring in the concurrent execution, i.e. inside the bean definition `activity_B`. Also `FallbackErrorHandler` is the common handler which the workflow uses across all activities.

#### CoffeeDetailsActivity.java

```java
public class CoffeeDetailsActivity extends BaseActivity {
    @Autowired
    CoffeeDetailsFetcherService coffeeDetailsFetcherService; // to fetch coffee details from DB

    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("Inside CoffeeDetailsActivity");
        System.out.println("Starting Coffee Sync");
        try {
            coffeeDetailsFetcherService.fetchCoffee();
            // rest of business logic here
            // NOTE: you can pass on the data from this activity to the rest of the activities through the "context", the implementation of which can be customized as per the needs. It can contains any data structures that will fit the business needs.
        } catch (Exception e) {
            System.out.println("Error in syncing coffee.");
        }
        System.out.println("Coffee Sync completed");
    }
}
```

#### ParallelDataActivity.java

```java
public class ParallelDataActivity extends ConcurrentActivity {
    public ParallelDataActivity(
            List<BaseActivity> parallelActivities,
            ErrorHandler errorHandler) {
        this.setParallelActivities(parallelActivities);
        this.setErrorHandler(errorHandler);
    }

    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("Inside GetParallelDataActivity");
    }
}
```

#### ManufacturerDetailsActivity.java

```java
public class ManufacturerDetailsActivity extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("Inside ManufacturerDetailsActivity");
        // logic to get the manufacturer details and append it with the coffee details present in the context
    }
}
```

#### PriceCalculatorActivity.java

```java
public class PriceCalculatorActivity extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("Inside PriceCalculatorActivity");
        // logic to tailor the price on various factors like place of origin, manufacturer details, profit margin, etc.
    }
}
```

#### CustomerPreferenceActivity.java

```java
public class CustomerPreferenceActivity extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("Inside CustomerPreferenceActivity");
        // logic to integrate the information as per customer preferences according to the requirements
    }
}
```

#### CombineCoffeeDetailsActivity.java

```java
public class CombineCoffeeDetailsActivity extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("Inside CombineCoffeeDetailsActivity");
        // logic to get all the data from the previous activities and combine it as per the needs
    }
}
```

#### SaveActivity.java

```java
public class SaveActivity extends BaseActivity {
    @Override
    public void execute(WorkflowContext context) throws Exception {
        System.out.println("Inside SaveActivity");
        // logic to save the processed information so that it can be sent to the customers at an appropriate time
    }
}
```

#### ThisWorkflowContext.java

```java
public class ThisWorkflowContext implements WorkflowContext {
    // as mentioned earlier, the data structures inside the context are totally customizable 
    // some example are:
    private List<CoffeeDetail> coffeeDetails;
    private List<ManufacturerDetail> manufacturerDetails;
    // note that this data is in-memory and is used to pass to other activities.
    // data used locally in one activity need not be present inside the context.
    
    // getters and setters for above...

    @Override
    public AtomicBoolean shouldWorkflowStop() {
        return true; // true as we cannot tolerate any errors in any activity, and would ideally like the workflow to be stopped in case of errors
    }

    @Override
    public void setSeedData(Object seedObject) {
        // here we can seed the workflow with any external data if we want, and then utilize the seed data in any of the activities
    }
}
```

#### ParallelErrorHandler.java

```java
public class ParallelErrorHandler implements ErrorHandler {

    @Override
    public boolean handleError(WorkflowContext context, Exception exception) {
        return false;
    }

    @Override
    public void setBeanName(String name) {

    }
}
```

#### FallbackErrorHandler.java

```java
public class FallbackErrorHandler implements ErrorHandler {
    @Override
    public boolean handleError(WorkflowContext context, Exception exception) {
        return false;
    }

    @Override
    public void setBeanName(String name) {

    }
}
```

### Starting Workflow Execution

Starting a workflow execution is as simple as creating a _WorkflowEngine_ and seeding it with:
 * the workflow to be run
 * any seed data to be sent from the outside world

`DefaultWorkflowEngine` is a default implementation of _WorkflowEngine_ provided inside the Easy-Workflow JAR.

```java
class WorkflowController {
    @Autowired
    Workflow coffeeNewsletterWorkflow;
    
    DefaultWorkflowEngine engine = new WorkflowEngine();

    public void generateCoffeeNewsletter() {
        WorkflowReport report = engine.run(
            coffeeNewsletterWorkflow, 
            null // replace with seed data you want to send to the workflow 
        );   
        
        // retrieve status of workflow using:
        report.getStatus();
        
        // retrieve completion/failure message using:
        report.getMessage();
        
        // retrieve Throwable object in case of an error using:
        report.getError();

        // access workflowContext using:
        report.getWorkflowContext();
        // above makes it possible to use context for sending the processed data to the outside workflow
    }
}
```

## The Inabilities

Just like any software is not suitable for all the needs, Easy-Workflow is no exception. Though it is totally extendable for some more scenarios.

### State Persistence

Easy-Workflow maintains an in-memory state for now. This can be persisted after every activity which can reap up more benefits like:

* restarting the workflow mid-way
* monitoring execution and analysing at a more granular level

### Manual Intervention

Easy-Workflow is not suitable for workflows requiring manual intervention in some of the activities like pausing -> noti
fying stakeholders -> seeking approvals -> then continuing the rest of the workflow on getting the approvals.

### Distributed Workflow

Easy-Workflow runs the workflow on a single machine as of now. So to deal with larger data, it can be enhanced so that the activities run on different machines and the workflow can be controlled in a more distributed fashion.

Having said that, enhancing Easy-Workflow to solve for distributed use-cases steals our initial purpose: it's not easy anymore! :)
   
## Final Words

Do you now see how we were able to achieve separation of concerns in a concise and organized manner? Personally, my OCD with clean code definitely got much better when I implemented this to solve one of very complex business use-cases at work!
