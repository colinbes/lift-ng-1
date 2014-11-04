# lift-ng

lift-ng is the most powerful, most secure AngularJS backend available today.

The design philosophy of **lift-ng** is to capture the spirit of both [Lift](http://liftweb.net) and [AngularJS](http://docs.angularjs.org/guide/overview) into one package.
The result is a secure-by-default framework facilitating powerful and robust client/server interactions for building today's modern web applications.
By utilizing Scala's powerfully-flexible language, **lift-ng** provides a DSL that is natural to use for AngularJS developers and Scala developers alike.

Lift as a backend should appeal to AngularJS developers for the following reasons:
* Lift's templating is unlike most web application frameworks in that it is plain HTML.
There is no massaging of your Angular templates to make the framework happy.
* The approach of manipulating the templates on the server by Lift is similar to how you manipulate them on the client with AngularJS.
Hence you can manipulate the DOM at the time you know the information, while on the client or earlier while on the server.
* Lift is not MVC.
No need to navigate another MVC framework while building with Angular's MVC approach.
* Lift is excellent at slinging HTML and JS.
This is precisely what an Angular application needs for a backend.
* Security is handled for you, making it virtually impossible to have your http endpoints successfully attacked.
(More on [Lift's security](http://seventhings.liftweb.net/security))
* Your application will be built on the rock-solid JVM as a time-tested Java servlet on the most mature Scala web framework.

AngularJS as a front end should appeal to Scala and Lift developers for the following reasons:
* JQuery is a pain for non-trivial applications.
* AngularJS does a fantastic job of managing complex client-side interactions for you.
* With **lift-ng** in particular, you get to utilize Lift's reactive features such as asynchronous comet updates to the client and returning `LAFuture[T]` results for services.

**lift-ng** has three major feature areas (click the respective link for details and usage examples):
* [Client-Initiated Service Calls](#client-initiated-service-calls): Write secure services in the Scala DSL which can be invoked by the client.
* [Server-Initiated Events](#server-initiated-events): Send events to `$rootScope` or a given `$scope` easily via familiar AngularJS-like calls to `broadcast` and `emit`.
* [Client-Server Model Binding](#client-server-model-binding): Define your model, declare your scope, and assign a name.
Then let **lift-ng** keep your data in sync between the client(s) and server.

## Tutorials

* [15-minute Chat with lift-ng](https://www.youtube.com/watch?v=PQA6829cRy8): Screencast by Lift committer/lift-ng contributor [Joe Barnes](https://github.com/joescii) briefly introducing Lift, AngularJS, and lift-ng.
* [ScalaDC (Scala + Lift + Angular) is Magic](http://www.youtube.com/watch?v=VH-S2UDN-NQ): Screen cast by original lift-ng contributor [Doug Roper](https://github.com/htmldoug) featuring Scala, Lift, AngularJS, ponies, magic, spastic keyboard pounding, and the first incarnation of lift-ng.

## Jump Start

The quickest jump start you can get on **lift-ng** is via the [giter8 template](https://github.com/joescii/lift-ng.g8).
Otherwise, you should first [get started with Lift](http://liftweb.net/getting_started) and configure the project to include the **lift-ng** module as outlined below.

## Configuration

You can configure an existing Lift project to use **lift-ng** manually.
Add `lift-ng` as a dependency in your `build.sbt` or `Build.scala` as appropriate.
Optionally add `lift-ng-js` as a dependency if you would like us to manage the delivery of your AngularJS library javascript files.

```scala
libraryDependencies ++= {
  val liftVersion = "2.5.1" // Also supported: "2.6" and "3.0"
  val liftEdition = liftVersion.substring(0,3)
  val ngVersion = "1.3.0"  // If using lift-ng-js as recommended
  Seq(
    // Other dependencies ...
    "net.liftmodules" %% ("ng_"+liftEdition)    % "0.5.6"            % "compile",
    "net.liftmodules" %% ("ng-js_"+liftEdition) % ("0.2_"+ngVersion) % "compile"
   )
}
```

And invoke `Angular.init()` in your `Boot` class (shown here with the default values).

```scala
package bootstrap.liftweb

class Boot {
  def boot {
    // Other stuff...
    
    net.liftmodules.ng.Angular.init(
      // Set to true if you plan to use futures. False otherwise to avoid an unneeded comet
      futures = true,

      // Set to the CSS selector for finding your apps in the page.
      appSelector = "[ng-app]",

      // Set to true to include a script tag with the src set to the path for liftproxy.js.
      // Set to false if you want to handle that yourself by referring to the path in
      // net_liftmodules_ng.
      includeJsScript = true
    )
  }
}
```

If you want to handle the downloading of javascript assets yourself with a library such as [head.js](http://headjs.com/), then you should initialize with `includeJsScript = false`.
This will prevent our `Angular` snippet from including the `liftproxy.js` file.
Instead, you can use the global `ng_liftmodules_ng` object to include the file yourself.
You can get the full path to the `liftproxy.js` file via `net_liftmodules_ng.path` or get just the lift-ng version alone with `net_liftmodules_ng.version`.

## Supported Versions

**lift-ng** is built and released to support Lift edition 2.5 with Scala versions 2.9.1, 2.9.2, and 2.10; Lift edition 2.6 with Scala versions 2.9.1, 2.9.2, 2.10, and 2.11; and Lift edition 3.0 with Scala version 2.11.
This project's scala version is purposefully set at the lowest common denominator to ensure each version compiles.
Automated testing is performed against the latest 2.10/2.5, 2.10/2.6, 2.11/2.6, and 2.11/3.0 Scala/Lift versions for each release of **lift-ng**.

## Usage

Below are usage examples of each of the major features of **lift-ng**.
Be sure to check out the aforementioned [sample project](https://github.com/htmldoug/ng-lift-proxy) or the [test project](https://github.com/joescii/lift-ng/tree/master/test-project) for fully functional examples.

### Client-Initiated Service Calls

Most AngularJS backends provide RESTful http endpoints for the application to receive data from the server.
**lift-ng** is certainly no exception by providing a server-side DSL for creating services.
Compare the following AngularJS factory

```javascript
angular.module('lift.pony', [])
  .factory("ponyService", function() {
    return {
      getBestPony: function(arg) {
        // Return the best pony (client-side)
        return BestPony;
      }
    };
  });
```

... to this **lift-ng** DSL in Scala

```scala
angular.module("lift.pony")
  .factory("ponyService", jsObjFactory()
    .jsonCall("getBestPony", (arg) => {
      // Return the best pony (server-side)
      Full(BestPony)
    })
  )
```

Both will create an angular module named `lift.pony` with a service named `ponyService` with a function named `getBestPony`, yet the former runs on the client-side, and the latter runs on the server-side.

To flesh out this example completely, you will define a [Lift snippet](http://exploring.liftweb.net/master/index-5.html) to provide the service to the resulting HTML.
Here, we have elected to create an object named `NgPonyService`, delivering the snippet via Lift's default snippet method name, `render`

```scala
object NgPonyService {
  def render = renderIfNotAlreadyDefined(
    angular.module("lift.pony")
      .factory("ponyService", jsObjFactory()
        .jsonCall("getBestPony", (arg) => {
          // Return the best pony (server-side)
          try {
            Full(BestPony)
          } catch {
            case e:Exception => Failure(e.getMessage)
          }
        })
      )
  )
}
```

This `renderIfNotAlreadyDefined` returns a `scala.xml.NodeSeq`.  Hence you will need to add script tags to your target HTML page(s) for the services as well as some plumbing from this module.

```html
<!-- The angular library (manually) -->
<script type="text/javascript" src="/scripts/angular.js"></script>

<!-- Or better yet, via lift-ng-js. (https://github.com/joescii/lift-ng-js) -->
<script data-lift="AngularJS"></script>

<!-- Prerequisite stuff the module needs -->
<script data-lift="Angular"></script>

<!-- The NgPonyService snippet defined above -->
<script data-lift="NgPonyService"></script>

<!-- Your angular controllers, etc -->
<script type="text/javascript" src="/scripts/pony.js"></script>
```

The resulting angular service returns a `$q` promise (see [AngularJS: ng.$q](http://docs.angularjs.org/api/ng.$q)).
When you call the service, you register callbacks for success, error, and notify (not currently utilized).

```javascript
angular.module('pony', ['lift.pony'])
  .controller('PonyCtrl', function ($scope, ponyService) {
    $scope.onClick = function () {
      ponyService.getBestPony().then(function(pony) {
        // We have our pony!
        $scope.pony = pony;
      },
      function(err) {
        // No pony!
        $scope.error = err;
      },
      function(progress) {
        // We're still working on getting that pony...
        // Well, not really... we don't use this today...
      });
    };
  });
```

#### Mapping Box to Promise

Values requested from the client are always wrapped in a `net.liftweb.common.Box`.
These `Box[T]` values are mapped to their respective `$q` promises as follows:

`Full(value)` => A resolved promise with the given value.
`Empty` => A resolved promise with `undefined` value.
`Failure(msg)` => A rejected promise with the given message value.

#### No arguments, string arguments, or case class arguments

Just like with Lift's `SHtml.ajaxInvoke`, you can make a service which takes no arguments.  Hence we could have defined our `ponyService.getBestPony` like the following...

```scala
angular.module("lift.pony")
  .factory("ponyService", jsObjFactory()
    .jsonCall("getBestPony", {
      // Return the best pony (server-side)
      try {
        Full(BestPony)
      } catch {
        case e:Exception => Failure("No Pony!")
      }
    })
  )
```

Or we can accept a `String`...

```scala
angular.module("lift.pony")
  .factory("ponyService", jsObjFactory()
    .jsonCall("getPonyByName", (name:String) => {
      // Return the matching pony
      try {
        Full(BestPony)
      } catch {
        case e:Exception => Failure("No Pony!")
      }
    })
  )
```

Finally, perhaps most importantly, we expect a case class to be sent to the server.
Note that the case class must extend `NgModel` for this to work.
(Read more about models [here](#model-objects))

```scala
case class Pony (name:String, img:URL) extends NgModel

angular.module("lift.pony")
  .factory("ponyService", jsObjFactory()
    .jsonCall("setBestPony", (pony:Pony) => {
      // Nothing to return
      Empty
    })
  )
```

#### Multiple function calls

All of the above functions can be part of the same service...
```scala
angular.module("lift.pony")
  .factory("ponyService", jsObjFactory()
    .jsonCall("getBestPony", {
      // Return the best pony (server-side)
      Full(BestPony)
    })

    .jsonCall("getPonyByName", (name:String) => {
      // Return the matching pony
      Full(MyPony)

    .jsonCall("setBestPony", (pony:Pony) => {
      // Nothing to return
      Empty
    })
  )
```

#### Futures
All of the examples thus far have assumed the value can be calculated quickly without expensive blocking or asynchronous calls.
Since it is quite common to perform expensive operations or call APIs which return a `Future[T]`, it is important that **lift-ng** likewise supports returning an `LAFuture`.

The same signatures for `jsonCall` are supported for futures:

```scala
angular.module("lift.pony")
  .factory("ponyService", jsObjFactory()
    .future("getBestPony", {
      // Create the future
      val f = new LAFuture[Box[Pony]]()
      // Do something to get the result
      futurePonyGetter(f)
      // Return the future that will contain a Pony
      f
    })

    .future("getPonyByName", (name:String) => {
      // Return a future containing the matching pony
      futurePonyGetter(name)

    .future("setBestPony", (pony:Pony) => {
      // Nothing to return
      val f = new LAFuture[Box[Pony]]()
      f.satisfy(Empty)
      f
    })
  )
```

Because the underlying Lift library does not currently support returning futures for AJAX calls (as of 2.5.1), we had to circumvent this limitation by utilizing comet.
As a result, if you want to utilize futures in your angular app, we must be able to locate your app in the DOM.
By default, we look for any elements containing the `ng-app` attribute.
This can be overridden in the `Angular.init()` call via the `appSelector` property.
This allows us to hook in to your app via comet and send messages asynchronously back to the lift-proxy service.

#### Testing
Testing services provided by **lift-ng** with Jasmine (etc) can be accomplished in the same manner as you would test any Angular service.

```javascript
describe("pony", function(){
  // Service mock
  var ponyService = {};

  // Handle to the rootScope
  var rootScope = {};

  // Handle to the scope
  var scope = {};

  // Set up the pony module
  beforeEach(function(){module('pony');});

  // Mock out the service
  beforeEach(function() {
    angular.mock.module(function($provide) {
      $provide.value('lift.pony', ponyService);
    });
  });

  // Build the mock with a $q promise
  beforeEach(inject(function($q) {
    ponyService.defer = $q.defer();
    ponyService.getBestPony = function() {
      return this.defer.promise;
    };
  }));

  // Create a controller for each test
  beforeEach(inject(function($rootScope, $controller) {
    rootScope = $rootScope;
    scope = $rootScope.$new();
    $controller('PonyCtrl', {
      $scope: scope,
      ponyService: ponyService
    });
  }));

  // Write a test
  it('should call the service when onClick is called', function() {
    // Before onClick, the pony will be undefined.
    expect(scope.pony).toBeUndefined();

    // Provide a pony to be returned
    var pony = {
      name: 'Doug',
      img: 'doug.jpg'
    };
    ponyService.defer.resolve(pony);

    // Simulate the click
    scope.onClick();

    // This call lets the $q callback happen
    rootScope.$digest();

    // Expect that pony has now been set.
    expect(scope.pony).toEqual(pony);
  });
});
```

#### Under the Services' hood
Unlike most AngularJS RESTful http backends, you have no further work to do to secure your application.
Rather than a fixed named endpoint, **lift-ng** dynamically creates an http endpoint for each service per page load.
The name given to the end points is a securely-randomized number that is difficult to predict for an attacker attempting to utilize cross-site scripting techniques for instance.

In regards to testing the server-side code, we recommend implementing your service logic in a standalone module which handles all of the complex business logic.
Then utilize **lift-ng** to merely expose those services to your angular app.
This way your business logic is decoupled from **lift-ng** and easily testable.

#### Non-AJAX
Sometimes the value you want to provide in a service is known at page load time and should not require a round trip back to the server.
Typical examples of this are configuration settings, session values, etc.
To provide a value at page load time, just use `JsonObjFactory`'s `string`, `anyVal`, or `json` methods.

```scala
angular.module("StaticServices")
  .factory("staticService", jsObjFactory()
    .string("string", "FromServer1")
    .anyVal("integer", 42)
    .json("obj", StringInt("FromServer2", 88))

  )
```

The above produces a simple service equivalent to the following JavaScript
```javascript
angular.module("StaticServices",["lift-ng"])
  .factory("staticService", function(liftProxy) { return {
    string:  function() {return "FromServer1"},
    integer: function() {return "42"},
    obj:     function() {return {str:"FromServer2",num:88}}
  }});
```

### Server-Initiated Events

Now we can take a look at how to utilize Lift's comet support to asynchronously send angular updates from the server

First you should write a new class which extends the `AngularActor` trait, which is a sub-trait of Lift's `CometActor`.
Thus you can do anything you can normally with a `CometActor`, as well as get access the `$scope` where the actor is defined in the DOM and the `$rootScope` of the angular application.
Currently we support `$emit`, `$broadcast`, and assignment of arbitrary fields on the given scope object.

```scala
class CometExample extends AngularActor {
  override def lowPriority = {
    case ("emit", msg:String) => rootScope.emit("emit-message", msg)
    case ("emit", obj:AnyRef) => rootScope.emit("emit-object",  obj)
    
    case ("broadcast", msg:String) => scope.broadcast("emit-message", msg)
    case ("broadcast", obj:AnyRef) => scope.broadcast("emit-object",  obj)

    case ("assign", msg:String) => rootScope.assign("my.str.field", msg)
    case ("assign", obj:AnyRef) => scope.assign("my.obj.field", obj)
  }
}
```

Now add the comet actor into your HTML DOM within the scope you wish to belong to within the `ng-application`.

```html
<div ng-app="ExampleApp">
  <div data-lift="comet?type=CometExample"></div>
  <!-- other stuff -->
</div>
```

Then do whatever you need in your angular application to listen for events, watch for changes, etc.

```javascript
angular.module('ExampleApp', ['lift-ng'])
.controller('ExampleController', ['$rootScope', '$scope', function($rootScope, $scope) {
  $rootScope.$on('emit-message', function(e, msg) {
    $scope.emitMessage = msg;
  });
  $rootScope.$on('emit-object', function(e, obj) {
    $scope.emitObject = obj;
  });
  $scope.$on('broadcast-message', function(e, msg) {
    $scope.broadcastMessage = msg;
  });
  $scope.$on('broadcast-object', function(e, obj) {
    $scope.broadcastObject = obj;
  });
  $rootScope.$watch('my.str.field', function(e, msg) {
    // ...
  });
  $scope.$watch('my.obj.field', function(e, obj) {
    // ...
  });
}]);
```

Note that messages sent prior to a page bootstrapping the Angular application will be queued up and released in order once the application is ready.
The retry interval defaults to 100 milliseconds and can be configured in your Lift application's props files with the `net.liftmodules.ng.AngularActor.retryInterval` Int property.

### Client-Server Model Binding
Just as Angular provides declarative 2-way binding between the model and view with automatic synchronization, **lift-ng** features binding of a model between the client and server.
To take advantage of this feature, first create a model case class which extends `NgModel`.
(Read more about models [here](#model-objects))

```scala
case class Message(msg:String) extends NgModel
```

Then create binder in your `comet` package which extends `NgModelBinder` or `SimpleNgModelBinder`.
(The latter is a conveniently-constructed form of the former)
By default, your binder will be scoped per-request/per-page-load.
Mix in `SessionScope` to cause your binder's state to persist among each page load of a user's session.
You must specify the direction you want to bind data by mixing in `BindingToClient`, `BindingToServer`, or both to achieve two-way synchronization.
The following example establishes a two-way binding existing in the session scope (identical to the original 0.5.0 release which featured `BindingActor`)

```scala
package org.myorg.comet

class MessageBinder extends SimpleNgModelBinder[Message] (
  "theMessage",       /* Name of the $scope variable to bind to */
  Message("initial"), /* Initial value for theMessage when the session is initialized */
  { m:Message =>      /* Called each time a client change is received */
    m
  },
  1000                /* Milliseconds for client-sync delay (see Optimizations below) */
) with BindingToClient with BindingToServer with SessionScope
```

Once defined, add the binder to the scope you want this model to exist in.

```html
<div ng-app="MyLiftNgApp">
  <div ng-controller="MyController" data-lift="Angular.bind?type=MessageBinder">
    <input type="text" ng-bind="theMessage.msg"/>
    <div>{{theMessage.msg}}</div>
  </div>
</div>
```

If you have multiple binds, you can specify them in `types` via a comma-delimited list:

```html
<div ng-app="MyLiftNgApp">
  <div ng-controller="MyMultipleBinds" data-lift="Angular.bind?types=Binder1,Binder2">
    <!-- Other stuff... -->
  </div>
</div>
```

If you mixed in `BindingToServer`, you will get state updates for the client (or _clients_ if using `SessionScope`) via `onClientUpdate`:

```scala
class MyBinder extends NgModelBinder[MyModel] with BindingToServer {
  override val onClientUpdate = { fromClient =>
    println(s"We go a model update from the user! $fromClient")
  }

  // ...
}
```

Since watching scope variables on the client can produce a flood of changes (e.g. each character entered in a text box will generate a change event), changes are queued up and sent after no more changes are detected for 1000 millis.
The fourth argument to the `SimpleNgModelBinder` constructor or an override of `clientSendDelay` in `NgModelBinder` allows you to tweak this delay to your liking.

If you mixed in `BindingToClient`, you can update your clients (if using `SessionScope`) by sending the binder a message with the model:

```scala
val newMessage = Message("Updated!")

// Find the binder
for {
  session <- S.session
  binder <- session.findComet("MessageBinder")
} { binder ! newMessage } // Send the new model
```

#### Optimizations
Mix in the `BindingOptimizations` trait to reduce the network overhead.
Changes to a bound model on the server will be communicated to the client by sending only a diff.

Currently the client sends the entire model back to the server on change even when mixing `BindingOptimizations`.
We hope to one day support sending only the diff just like we do when sending from the server.

Arrays are not correctly supported yet.
A client-side change to an array will append to the array on the server rather than replacing the respective values based on the index.

Removing stuff from a model on the server does not transmit to the client yet.
Only changes or additions to the model will be synced up to the client.

#### Memory consumption
Depending on the mixins utilized, you will store up more memory on the server when using an `NgModelBinder`.
If utilizing `BindingOptimizations` or `SessionScope`, we must maintain the last known state of your model.
This allows us to (1) compare to any model changes provided and transmit only the diff from the server and (2) render new pages with the current state.

### Model objects
Any case class can be used as a model in **lift-ng**.
However, models which are sent to the server from the client must mix in the `NgModel` trait.
While models serialized to the client don't need this flag trait, but it is a good practice to include it to avoid errors as your application changes over time.
Note that objects contained in an `NgModel` instance also do not need this flag trait either.

#### Embedded Futures
In addition to data fields which serialize naturally to their equivalent JSON representation, any model can contain fields that are futures of type `net.liftweb.actor.LAFuture[Box[T]]` for an arbitrary `T <: Any`.
Such fields will be mapped to the client representation of the model as a promise from the `$q` angular service.
The future will be plumbed to the client-side promise automatically.
Any futures occurring in the model object graph will be serialized and plumbed.

For instance, given this Scala case class model:

```scala
case class MyModel (
  fastValue:String,
  slowValue:Future[Box[String]]
) extends NgModel
```

You will receive the following object on the client:

```javascript
var myModel = // However you get it from lift-ng
myModel.fastValue // A string
myModel.slowValue // A promise

myModel.slowValue.then(function(value){
  console.log('The value is '+value)
}
```

Once the `LAFuture` is satisfied, the result will be pushed up via comet to resolve/reject the promise according to the `Box` value.
The `Box` value is mapped with the same logic as with client-initiated service calls.
See [Mapping Box to Promise](#mapping-box-to-promise).

### i18n Internationalization
If your app doesn't require sophisticated internationalization capabilities (i.e., Java resource bundles will suffice), then you can inject your resource bundles as a service into your app.

Given a resource bundle named `bundleName.properties`:

```text
hello=Howdy!
goodbye=Goodbye, {0}!
```

Add it as a service available to your Angular app with this HTML:

```html
<script id="my-i18n_js"  data-lift="i18n?name=bundleName"></script>
```

Your bundle is made available via the `i18n` module with service/factory name coinciding with the bundle name.
In this example, the object will have a string field named `hello` and a function named `goodbye`:

```javascript
angular.module('ExampleApp', ['i18n'])
.controller('ExampleController', ['$scope', 'bundleName', function($scope, i18n) {
  $scope.hello = i18n.hello;
  $scope.goodbye = i18n.goodbye($scope.username);
}]);
```

You may also specify multiple bundle names.  Here we include the default Lift bundle:

```html
<script id="my-i18n_js"  data-lift="i18n?names=bundleName,lift"></script>
```

Each bundle is another service available in the `i18n` bundle.
Also notice in this example we show keys which aren't valid JavaScript identifiers are also available.

```javascript
angular.module('ExampleApp', ['i18n'])
.controller('ExampleController',
  ['$scope', 'bundleName', 'lift', function($scope, bundle, lift) {
    $scope.hello = bundle.hello;
    $scope.goodbye = bundle.goodbye($scope.username);
    $scope.lostPasswd = lift["lost.password"];
}]);
```

For more details about this resource bundle object, see [j2js-i18n](https://github.com/joescii/j2js-i18n).

## Scaladocs

The latest version of scaladocs are hosted thanks to [cloudbees](http://www.cloudbees.com/) continuous integration services.  There should not be any differences among the supported versions of Scala.  Nonetheless all are listed here for good measure.
* [Scala 2.10](https://liftmodules.ci.cloudbees.com/job/lift-ng/ws/target/scala-2.10/api/index.html#package)
* [Scala 2.9.2](https://liftmodules.ci.cloudbees.com/job/lift-ng/ws/target/scala-2.9.2/api/index.html#package)
* [Scala 2.9.1-1](https://liftmodules.ci.cloudbees.com/job/lift-ng/ws/target/scala-2.9.1-1/api/index.html#package)
* [Scala 2.9.1](https://liftmodules.ci.cloudbees.com/job/lift-ng/ws/target/scala-2.9.1/api/index.html#package)

## Community/Support

Need help?  Hit us up on the [Lift Google group](https://groups.google.com/forum/#!forum/liftweb).  We'd love to help you out and hear about what you're building.

## Contributing

As with any open source project, contributions are greatly appreciated.  If you find an issue or have a feature idea, we'd love to know about it!  Any of the following will help this effort tremendously.

1. Issue a Pull Request with the fix/enhancement and unit tests to validate the changes.  OR
2. Issue a Pull Request with failing tests in the [test-project](https://github.com/joescii/lift-ng/tree/master/test-project) to show what needs to be changed OR
3. At a minimum, [open an issue](https://github.com/joescii/lift-ng/issues/new) to let us know about what you've discovered.

### Pull Requests

Below is the recommended procedure for git:

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

Please include as much as you are able, such as tests, documentation, updates to this README, etc.

### Testing

Part of contributing your changes will involve testing.
The [test-project](https://github.com/joescii/lift-ng/tree/master/test-project) sub-directory contains and independent sbt project for thoroughly testing the **lift-ng** module via selenium.
At a minimum, we ask that you run the tests with your changes to ensure nothing gets inadvertently broken.
If possible, include tests which validate your fix/enhancement in any Pull Requests.

## Wish list

Here are things we would like in this library.  It's not a road map, but should at least give an idea of where we plan to explore soon.

* Correctly support client-side changes to models containing arrays when mixing `BindingOptimizations` into an `NgModelBinder`.
* Support removing fields in models bound with an `NgModelBinder` with `BindingOptimizations` mixed in.
* Retain diffs in `NgModelBinder`. Allow server to reject client-side changes.
* Support Lift's Record with `NgModel`.
* Support dynamically-loaded templates for routing frameworks. (See [Issue 7](https://github.com/joescii/lift-ng/issues/7))
* Produce an error message when an attempt is made to use a model which does not extend `NgModel`. (Currently this silently fails)
* Support handling parameters of type `json.JValue`.
* Support returning values of type `JsExp`.
* Support server comet pushes to client via services.
* Initial value/first resolve value for services.  The reason for providing a first value will allow the page load to deliver the values rather than require an extra round trip.
* `AngularActor.scope.parent` support

## Change log

* *0.6.0*: Introduction of [embedded futures](#embedded-futures).
All of your angular modules now must directly or indirectly depend on the `lift-ng` module.
* *0.5.6*: Bug Fix: Strings pushed to the client are now properly escaped.
Prior to this fix, a string containing illegal characters such as a newline would be silently discarded.
* *0.5.5*: Further decomposed `NgModelBinder`, separating the transmission optimizations into the `BindingOptimizations` mixin.
With this update, an `NgModelBinder` should work for all cases by default.
* *0.5.4*: **BREAKING CHANGE:** Renamed `BindingActor` to `NgModelBinder`.
Decomposed `NgModelBinder` so it is possible to specify:
  * `BindingDirection` by mixing in `BindingToClient`, `BindingToServer`, or both for 2-way binding
  * `BindingScope` which defaults to per-request (i.e. per page load) and can be scoped to the session like the original `BindingActor` behavior by mixing in `SessionScope`
* *0.5.3*: Fixed handling of `NgModelBinder` initial values.
Fixed usage of `CometListener` with a 2-way session-scoped `NgModelBinder` by reversing the order in which the named/unnamed comet actors are rendered.
Fixed support for Lift 3.0-SNAPSHOT.
Enhanced automated testing to cover 2.10/2.5, 2.10/2.6, 2.11/2.6, and 2.11/3.0 Scala/Lift versions.
* *0.5.2*: Resolved [Issue #5](https://github.com/joescii/lift-ng/issues/5), where the deployment context path appeared twice in the path to the `liftproxy.js` resource.
Dropped support for Lift 3.0 compiled against Scala 2.10.
* *0.5.1*: Corrected a bug exposed by 2-way binding that our [early-arrival mechanism](https://github.com/joescii/lift-ng/issues/1) to not work if `angular.js` files are specified at the end of the HTML.
Made it possible to add `NgModelBinders` in the HTML templates without introducing an extra element.  Thanks to [Antonio](https://twitter.com/lightfiend) for [the suggestion](https://groups.google.com/forum/#!topic/liftweb/1SJ6YNzpBEw)!
* *0.5.0*: Introduction of 2-way client/server `NgModel` binding.  Added support for Scala 2.11 against Lift editions 2.6 and 3.0.
* *0.4.7*: Updated to work on pages that are in subdirectories.  See [Pull Request #4](https://github.com/joescii/lift-ng/pull/4).  Thank you [voetha](https://github.com/voetha) for the contribution!
* *0.4.6*: Minor correction to resolution for [Issue #1](https://github.com/joescii/lift-ng/issues/1) to correctly allow messages to begin dequeuing without waiting for a new message. 
Added `includeJsScript` parameter to `Angular.init()` to give developers the ability to download the `liftproxy.js` their own way, such as via [head.js](http://headjs.com/).
Updated closure compiler. 
* *0.4.5*: Now queues and releases async (`AngularActor`) messages arriving prior to Angular bootstrapping, resolving [Issue #1](https://github.com/joescii/lift-ng/issues/1).
* *0.4.4*: Fixed the last version, which would serve the same i18n locale resource for every requested locale.
* *0.4.3*: Enhanced i18n service to be served restfully, allowing the browser to cache the service if it has not changed. Dropped 2.9.1-1 support. Began compiling 2.10 with 2.10.4.
* *0.4.2*: Reverted a change made in 0.4.0 which allows more flexibility in the placement of angular services in the DOM, fixing part 2 of [Issue #2](https://github.com/joescii/lift-ng/issues/2)
* *0.4.1*: Now works for web apps running in a non-root context, fixing [Issue #2](https://github.com/joescii/lift-ng/issues/2).
* *0.4.0*: Now only requires `Angular` snippet in your template(s).
* *0.3.1*: Added i18n service.
* *0.3.0*: Implemented support for a factory/service to return an `LAFuture[Box[T]]`
* *0.2.3*: Implemented `string`, `anyVal`, and `json` on `JsonObjFactory` to allow providing values which are known at page load time and do not otherwise change.
* *0.2.2*: Implemented `AngularActor.assign` for assigning scope variables. `Failure(msg)` message is sent to the client for `$q.reject`. Changed interpretation of `Empty` to mean `Resolve` rather than `Reject`.
* *0.2.1*: Implemented `scope.broadcast` and `scope.emit` for `AngularActor`
* *0.2.0*: Introduction of `AngularActor` featuring `rootScope.broadcast` and `rootScope.emit` as the first comet-backed features
* *0.1.1*: First working release
* *0.1*: First release featuring AJAX services invoked from the client

## License

**lift-ng** is licensed under [APL 2.0](http://www.apache.org/licenses/LICENSE-2.0).

Copyright 2014 net.liftweb

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

