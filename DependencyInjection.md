
Dependency Injection
====================

What is Dependency Injection?
-------------------------------
Dependency injection is a style of object configuration in which an objects fields and collaborators are set by an external entity. In other words, objects are configured by an external entity. Dependency injection is an alternative to having the object configure itself. The primary goals are reusability and maintenance of software through loose binding and to simplify testing.

### Single responsibility principle
The single responsibility principle is a programming principle that states that every module or class should have responsibility over a single part of the functionality provided by the software. Additionally, responsibilities should be entirely encapsulated by the class. 
"A class should have only one reason to change." – A famous quote from Robert C. Martin which summarizes the intention of this principle very well.
### Dependency inversion principle
The Dependency Inversion Principle (DIP) states that high level modules should not depend on low level modules; both should depend on abstractions. Abstractions should not depend on details.  Details should depend upon abstractions.



![alt text](/example_light.png "Inversion principle exaple: light switch")

This example shows the dependencies between a light and its switch. The class light proofs whether the light is on or off. The class Light switch implements the business logic for pressing the button. As a consequence, Light switch controls the process flow and uses Light. If methods of Light get renamed Light switch must change, too. So, Light switch should be a higher-level module than Light.
The problem can be solved with an interface which must be implemented by Light switch. That’s how dependencies got reversed and the example follows the Dependency inversion principle.
### Basic implementations
#### Constructor Injection
Constructor injection means that dependencies from other classes are provided by constructors.
```java
class Client {
private Service service;
 	// Constructor 
 	 public Client(Service service) { 
 	      this.service = service; 
 	} 
}
class Injector {
	void method() {
	final Service service = …;
	final Client client = new Client(service);
	} 
}
```

#### Setter Injection
Using setter injection, the dependent class provides methods for injecting dependencies. This method is called with every injection.
``` java
class Client {
private Service service;
 	 // Setter method 
  	public void setService(final Service service) { 
  	this.service = service; 
  	} 
}
class Injector {
	void method() {
		final Client client = …;
		final Service service = …;
		client.setService(service);
	} 
}
```

#### Interface Injection
With interface injection the class defines an interface which needs to be implemented by dependent classes. This is simply the client publishing a role interface to the setter methods of the client's dependencies. It can be used to establish how the injector should talk to the client when injecting dependencies.
``` java
// Service setter interface. 
public interface ServiceSetter { 
 	 public void setService(final Service service); 
} 
 	
public class Client implements ServiceSetter {
 
 	private final Service service;  
 	    
 	public void setService(final Service service) { 
 	      this.service = service; 
 	} 
} 

class Injector {
	void method() {
		final Service service = …;
		final ServiceSetter servicesetter = …;
		servicesetter.setService(service);
	} 
}

```


Basic Java injector
-------------------

To understand how injection frameworks work it can be useful to build a small injector of your own. This way it is far easier to understand what these frameworks are doing for you and how much time they can save.

### No Dependency Injection

We'll start with a simple Application that uses a UserService to load the data of users. At the beginning everything works expected:
- Application initializes a new UserService and uses it to execute the method "getAllUsers"
- UserService initializes a new DBService to get the data from the database

The application uses the Service like:
``` java
//Using database
UserService userService = new UserService();
userService.getAllUsers();
```

This seems not very sophisticated but it could work. Sadly there are two problems: 
- Testing the UserService without the DBService is not possible.
- Using a API to get the data requires another field in the UserService or another service


### Using interfaces for DI
A simple way to get the dependencies out of the classes is to use interfaces.

In this case an interface SourceInterface is created. It is implemented by the DBService and the newly added APIService. But still the service creates the instances. Therefore the creation of a new Source is moved to the application. The created SourceService is now a constructor parameter of UserService.

Now the Application is able to decide if the user data should be provided by the API or the database by creating a new UserService with either an APIService or an DBService. The User service only knows the interface. In this way in a test of UserService is possible by using a MockService implementing the interface.

The application uses the UserService now like this:
``` java
//Using database
SourceInterface dbService = new DBService();
UserService dbUserService = new UserService(dbService);
userService.getAllUsers();
//Using API
SourceInterface apiService = new APIService();
UserService apiUserService = new UserService(apiService);
userService.getAllUsers();
```

### Using a simple injector
The currently used application has to know which Sources there are, how to initialize them and if those would have dependencies of their own the application had to initialize those too.

For this reason an injector should build the UserService. While the Application only has to initialize the injector and request the Service, the injector will take care of the dependencies.

To be able to switch between the sources two injector classes are created to build a UserService using the DBService or the APIService. Each of them contains a method that returns a UserService. Let's call tem "DBUserInjector" and "APIUserInjector".

Now, the application could get the user data like this:

``` java
//Using database
DBInjector dbInjector = new DBInjector();
UserService dbUserService = dbInjector.getService();
dbUserService.getAllUsers();
//Using API
APIInjector apiInjector = new APIInjector();
UserService apiUserService = apiInjector.getService();
apiUserService.getAllUsers();
```




DI in Java
----------
Until now there is no java library for Dependency Injection. A multitude of programmer used the Spring-Framework for DI in Java. But because of some disadvantages like no type safety or lots of XML files many started using the Google Guice framework. 
But it’s also possible to implement DI in Java without frameworks. With the JSR-299 a new standard called „Contexts and DI for the Java EE platform “, short “CDI”, came along. This standard is extensive but the Weld documentation gives a good insight. 
The CDI standard allows Injection in variables but setter injection or constructor injection is also possible. The container build a new object with every injection. If it’s not necessary, you can use the @Singleton annotation.
CDI Example:
```java
public class Configuration
{
 	public String greetingMessage() { return "Willkommen
 	Lena, ähh, CDI "; }
} 
```

The class Configuration should represent information about the configuration which are used by other Java objects. Instead of programming a factory or a singleton a container should build and inject the object.

```java
import javax.inject.Inject;

public class Application
{
	@Inject Configuration configuration;

	public void start()
	{
	System.out.println(configuration.greetingMessage());
	}
} 
```

The class Application is the main program which has recourse to the class Configuration.
With the annotation @Inject a reference to the Configuration object should be injected. It’s possible to inject as many attributes as you like. Afterwards, the Application object is initialized correctly. The start() method is a self-chosen method that starts the program. Here you can resort to all injected attributes and the program will test it with a small console output.
Finally, the application must be started. In this point the CDI container differ a little bit. There are lots of possible versions you can find at Weld. We choose a class with a static main() method which initializes the Weld container an calls the start() method.

``` java
public static void main( String[] args )
{
	Application app = new 	Weld().initialize().instance().select(Application.class).get();

	app.start();
}
```
Frameworks
----------

### Google Guice

Google Guice is a lightweight open source framework for dependency injection released by Google. It provides support for DI using annotations to configure objects. With Guice classes can be bound programmatically to an interface, then injected into constructors, methods or fields using an @Inject annotation. Guice aims to make development and debugging easier and faster. So this framework embraces Java’s type safe nature and helps to reduce boilerplate code. Dependencies are managed with Java code instead of XML files. To compliment dependency injection, Guice supports Aspect Oriented Programming (AOP). This feature enables you to write code that is executed each time a matching method is invoked. It's suited for cross cutting concerns, such as transactions, security and logging. Additionally, it’s possible to set scopes for used objects. Scopes allow you to reuse instances: for the lifetime of an application (@Singleton), a session (@SessionScoped), or a request (@RequestScoped). By default, Guice returns a new instance each time it supplies a value (Default scope).
Working with Guice, you first have to bind the interface to the implementation. For a better testability a mock class is used. If a variable from the class Booking is found, an instance from MockBooking will be injected.

```java
class PlanerModule implements Module {

 	public void configure(Binder binder) {
 	binder.bind(Booking.class).to (MockBooking.class);
 	}
} 
```

The question where it should be injected is solved with the @Inject annotation. This annotation indicates that a Booking object must be injected. In this example shows a constructor injection. But Guice masters also method and field injection, whereby field injection cannot be recommended.


```java
class Planer {
 	private final Booking booking;

 	@Inject
 	public Planer(Booking booking) {
 	this.booking = booking;
 	}
} 
```

As far as all dependencies are implemented Guice is able to start working. Whilst starting the application it starts bootstrapping the root object. Furthermore, Guice masters the residual injection with recursion.  During this process a validation takes place which points out gaps or mistakes.

```java
class Urlaubsplanung {

	public static void main(String[] args){
	Injector injector = Guice.createInjector();
	Planer planer = injector.getInstance (Planer.class);
	// business logic
```



### Dagger

Another popular example of frameworks supporting DI is **Dagger**. While Dagger 1.x was created by Square, Google took over the development since 2.0. The following chapter is all about version 2 of this framework.

Dagger allows DI by using annotations to Java code, to generate the classes needed to provide dependencies. Unlike Guice Dagger works at **compile time**. In this way, you can be sure your software will not throw exceptions at runtime, because of your dependencies, if it compiled successfully. It is important to note, that Dagger is most popular when it comes to Android development. Anyway, this description will describe the basics of using dagger Java, to allow an easy start for most readers.

A good point to start are Daggers annotations, because they define the different elements of an Dagger project.

#### @Component
While the following elements define the dependencies, how and where to provide them ("dependency graph"), Components are the **public interface to your dependencies**. To define a Component, use an interface and the annotation "@Component":

```java
@Component
public interface MyApplicationComponent {
    public MyService getMyService();
}
```
While it is possible to use dependencies across Components, most of the time it is the best way to use new Components, if this parts of your application are independent.

Dagger will generate a class for your interface which is always named after our interface with a Dagger-prefix. So the class for MyApplicationComponent would be named "DaggerMyApplicationComponent". Note that you'll have to build your application before you are able to use this class. To access the defined class add the following code:

```java
MyApplicationComponent comp = DaggerApplicationComponent.builder().build;

MyService service = comp.getMyService();
```



#### @Provides

Of course Dagger has to know which dependencies exist, to generate the desired code for you. Therefore you'll have to **describe the dependencies**. This means you'll have to define methods providing your dependency and add a "@Provides" annotation to them.

```java
@Provides
MyDependency myDependency() {
    return new myDependency();
}
```
If the dependency you declared has another dependency, you can set it as parameter to your method and give it to your dependency. To enable Dagger to inject it into your first dependency, create a new Provide for your new dependency.

```java
@Provides
MyDependency myDependency(NewDependency newDep) {
    return new myDependency(newDep);
}

@Provides
NewDependency newDependency() {
    return new NewDependency()
}
```
In this way you can define how to create your classes and the required dependencies.

#### @Module
Thanks to modules you can now **group your dependencies**. In most cases it is a good way separate modules by concern. (e.g. NetworkModule)

To create a Module annotate a Java class with "@Module". Then you can place you're dependencies (@Provide) inside of this class.

```java
@Module
class MyModule {
    @Provides
    MyDependency myDependency(NewDependency newDep) {
        return new myDependency(newDep);
    }

    @Provides
    NewDependency newDependency() {
        return new NewDependency()
    }
}
```
The last step is to tell your Component about your new Module by adding this information to the @Component annotation. Only if this is done you can use your module's dependencies in your Component.

``` java
    @Component(modules = MyModule.class)
    public interface MyApplicationComponent {
```

Similar you can add one module to another module.
``` java
    @Module(includes = AnotherModule.class)
    class MyModule {
```


#### Scoping

With the current setup every time a dependency is initialized all it's dependencies are also newly initialized. For example if you have a network service using a http client, ever time you initialize the service the service will use a new instance of the HttpClient. This means a lot of overhead especially for large applications. While you can use @Singleton to define a scope, you can also create a custom scope.

To use a dependency as a Singleton just add @Singleton to it. A module containing a @Singleton annotated dependency has to be a Singleton by itself.

``` java
@Provide
@Singleton
MyDependency myDependency() {
```

You can also create a @interface and annotate it with "@Scope" and "@Retention" to define your own scope.

``` java
@Scope
@Retention(RetentionPolicy.CLASS)
public @interface MyApplicationScope {}
```
Now you can apply your scope to all elements by using @MyApplicationScope. For example your component:

 ``` java
 @MyApplicationScope
 @Component(modules = MyModule.class)
 public interface MyApplicationComponent {
```

Note that you'll have to add a module to the scope if you want to add it's dependencies to it.

#### Is there more?

There are far more features of Dagger to fit the generated code to your needs. You can define custom Scopes in different ways, use dependencies across components or  use qualifiers to use dependencies with the same name. Also you can use @Inject to inject dependencies into fields.

#### Conclusion
While Dagger's complexity raises fast as you dive deeper, it is not hard to start with it. Also are the error-log useful because,
- they are almost readable
- they are traceable in the generated code
- you get them at compile time. So if your application compiles you can be sure you won't get Dagger related problems at runtime for e.g. forgetting to provide a dependency.

On the other hand the documentation is not very helpful because of it's structure and length. This way you'll have to rely on multiple sources to learn how to handle your Dagger problems. 

If you want to start a new project with Dagger is important to keep Dagger in mind while developing the architecture of your software. This way you can take full advantage of this software. Also you should rethink if your project is large enough to benefit of Dagger.


Advantages/ Disadvantages of DI
------------------------------
### Advantages

Dependency injection:
- facilitates testing
- decreases coupling between a class and its dependency
- reduces complexity in classes
- allows an object to remove all knowledge of its concrete implementation
- reduces boilerplate code 
- allows independent or concurrent development

### Disadvantages

Dependency injection:
- can make code difficult to read
- Separation behavior from construction
- forces complexity to move out of classes 
- can encourage dependence on a DI framework 





