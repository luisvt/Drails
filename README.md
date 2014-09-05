#Drails

DART MVC Framework inspired on Grails. Easy to setup and part of the dart force framework!

##Setup the Server
To setup a server you just need to call the method `initServer`, for example:

    main() {
      initServer(); //Initialize a server that listen on localhost:4040
    }

If you want to define a different address or port for your server, you just need to pass the decided values. For example:

    main() {
      initServer(address: new InternetAddress('<someaddress>'), port: 9090);
    }

##Dependency Injection
The framework has a small dependency injection system that wires Classes with Postfix: Controller, Service, and Repository. For example, you can create a abstract class named 'SomeService':

    abstract class SomeService {
      String someMethod();
    }
    
and then we can create next implementation class:

    class SomeServiceImpl implements SomeService {
      String someMethod() => 'someMethod';
    }
    
in that way whenever a controller has a variable of Type SomeService and is annotated with @autowired the framework is going to inject SomeServiceImpl into the controller.

If you have a third level class that implements or extends SomeService this class is going to be used doubt to the level.

```
NOTE: Since Dart has support for Global Variables and Methods, I think that we don't need to add support for annotations @Component, @Value, or @Bean that are used on Spring MVC.
```

##Create Controllers
To create a controller you only need to append 'Controller' to the class name, for example:

    class PersonsController {
      //Methods
    }
    
###Methods and URIs
By convention a URI is created with the combination of the Controller name and the method name:

    class HiController {
      // method mapped to: Hi/index
      String index() => "hi";
      
      // method mapped to: Hi/get2
      String get2() => 'get2';
    }

Methods: get, getAll, save, saveAll, delete, and deleteAll are being ignored during the mapping, so that they only going to have the controller name

###Path Variables
By convention path variables are going to be the required arguments from the methods, for example:

    class HelloController extends HiController {
    
      // mapped to: Hello/getVar/{id}/{var1}  
      String getVar(int id, String var1) => 'get: $id, $var1';
      
    }

If the argument type is HttpRequest, HttpHeaders or HttpSession, this is not going to be mapped as path variable. In contrast, those arguments are going to be passed to the method from the current request.

###Query parameters
By convention query parameters are going to be mapped from the optional arguments, for example:

    class PersonsController extends HiController {
      
      // mapped to: persons?pageSize=25&pageNumber=2 or just persons
      String getAll({int pageSize: 20, int pageNumber: 1}) {
        return 'getAll: $pageSize, $pageNumber';
      }
    }

in the previous sample pageSize and pageNumber are optional, so users could or could not write those values during a new request
    
###Autowiring Variables
To autowire a variable by Class Type and Weight you only need to annotate that variable with @autowired (similar to Spring).

    class HelloController extends HiController {
      String someString;
      
      @autowired
      HelloService helloService;
      
      String index() => helloService.sayHello() + ' and from HelloController ' + super.index();
    }

###Create Rest Controllers

If you want to create a REST-API with controllers you could use next:
 
    class PersonsController {
    
      // GET persons/1
      Person get(int id) => persons[id];
  
      // GET persons
      List<Person> getAll() => persons.values.toList();
  
      // PUT persons/1 {jsonObject}
      Person save(int id, @RequestBody Person person) => persons[id] = person;
  
      // POST persons 
      //   RequestBody: {jsonObject} or POST persons [jsonList]
      Iterable<Person> saveAll(@RequestBody List<Person> persons) => 
        persons..forEach((person) {
        if(person.id == null) {
          person.id = this.persons.keys.last + 1;
        }
        this.persons[person.id] = person;
      });
  
      // DELETE persons/1
      void delete(int id) { persons.remove(id); }
      
      // DELETE persons 
      //   RequestBody: int or [intList]
      void delete(List<int> ids) { ids.forEach((id) => persons.remove(id)); }
    }
    
Although REST tells that POST and DELETE should only handle one resource, I decide to also handle multiple resources. This is because there are some cases where users need to do that, so in this way reducing the number of queries made to the server.

If you do a POST with id null the server does an INSERT but if the id is present it does an UPDATE.

##TODOs

* Create Dependency Injection Tests
* Create Controller Tests
* Create Generic Rest Controller which should be extended for other controllers
* Add Object Relational Mapping (ORM) or Domain Object Model (DOM) support
* Create Repositories or Store objects to connect to database
* Add file server
* Add Exceptions Handlers
* Handle Serialization Exception
* Handle bad request exception
* Add Angular Dart or Polymer Dart client side
* Handle Cyclic Reference parsing
* Handle hashcode, _ref or @id variable to deserialize objects that don't have ids (for inserting purposes).
 