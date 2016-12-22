
=============

learning notes
==============

## architecture

### modules
modules in an app should correspond to features
all angular apps have at least one module: the root module
* this is always named `AppModule`

modules are classes that have the `@NgModule` decorator
the decorator takes a single metadata object that describes:
* imports - classes exported from other modules used in component templates
* exports - declarations exposed to component templates in other modules
* providers - objects that create services avaliable to the whole app(?)
* declarations - 'view classes' that belong to this module
* bootstrap - root component that hosts other views, essential to launching
 * only the root module sets this property

modules in angular are completely different than javascript modules
they're used side-by-side and without distinction often
* this is very confusing

### libraries
angular ships as a collection of these libraries
import them using the javascript `import` statement
identify them with the `@angular` prefix
you can import decorators, library modules

importing a module isn't enough
* include in the `imports` property of that component's `NgModule` metadata
 * the `import` statement is the javascript module system
 * the `imports` metadata in `NgModule` is the angular module system
 * docs say "confusion yields to clarity with time and experience"

### components
hold the logic that controls and drives views
consists of a javascript class
angular takes care of instantiating them and all that

### templates
a property within components
made of html and angular-specific properties
very similar to other template languages
has capability to loop through things, interpolate
can also reference tags exported from other components
* `<hero-detail *ngIf="selectedHero" [hero]="selectedHero"></hero-detail>`

### metadata
most things in angular are regular javascript things
the 'angularness' of them is described in metadata, attached by decorators
decorators take configuration objects that describe relationships

### data binding
angular support different forms of data binding
this is one of the major sells to using a framework like this at all

`{{ }}` syntax binds from component to dom
`[hero]="componentVariable"` does the same
`(click)="selectHero(hero)"` calls a function in the component
`[(ng-model)]="property"` is the famous two-way binding

```
	<li>{{hero.name}}</li>
	<hero-detail [hero]="selectedHero"></hero-detail>
	<li (click)="selectHero(hero)"></li>
	<input [(ngModel)]="hero.name">
```
data binding is evaluated once per javascript event cycle
* evaluation begins at the root component, proceeds through children

### directives
components are a subset of directives, they just add some metadata
instruct angular on how to modify the dom when rendering templates
'structural directives' are things like `*ngFor`, and `*ngIf`
'attribute directives' look like html attributes `<input [(ngModel)] ...>`

### services
classes with a single purpose that do things for your application
they're just classes that your other stuff needs access to
services are 'injected' into classes using that pattern with the constructor

```
  constructor(
    private backend: BackendService,
    private logger: Logger) { }
```

services are where we put tasks when we want to keep them out of components
fetching data from a server, performing computation, etc.

### dependency injection
injection is handled by angular and described by the component's constructor
* `constructor(private service: HeroService) { }`
when you register a `provider` for a service, that provider handles injection
* it will return an instance of the service class if it exists
* if there is no instance of the service class, it will create one and return it
services added to the root module will be available everywhere
* this is usually desired, and usually the best way to go about it
* registering a provider with a component means new instances of the component
  get new instances of the service


## the root module

the root module defines how everything is put together
metadata contains three essential components
* imports - at least contains the `BrowserModule`
* declarations - a component to run
* bootstrap - the root component angular creates and inserts into `index.html`

### imports
the imports array contains all the modules used by the application
these should be only `NgModule` classes
javascript imports give access to symbols exported by other js modules
angular imports tell angular about angular modules (classes with `@NgModule`)

### declarations
lists every component that belongs to the app
only components are allowed
module classes are not allowed, nor serivces nor model classes

### bootstrap
'bootstrapping' creates the component listed in the bootstrap array
it inserts each created component into the dom
applications usually bootstrap only the single root component

bootstrapping usually occurs with a JiT compiler during development
other methods of bootstrapping are used with production environments

`platformBrowserDynamic().bootstrapModule(AppModule)`
* takes the `AppComponent` component from `AppModule`'s bootstrap array
* sets up `AppComponent` in the dom tag identified by by the component's selector


## displaying data

### interpolation
`{{ }}` takes properties from the component and interpolates them into the template
`*ngFor` can repeat a tag for any iterable object in the component
this stuff was covered in the tutorial
`*ngIf`, whatever


## user input

### binding to events
like this
`<button (click)="onClickMe()">click me</button>`
* `(click)` identifies the 'target' of the binding
* `onClickMe()` is the 'template statement' evaluated when the event fires
 * these statements are evaluated from within the context of their component

angular puts the event object into a special varible called `$event`
* `<input (keyup)="onKey($event)">`

### template reference variables
provide direct access to an element from within the template

```
	<input #box (keyup)="0">
	<p>{{box.value}}</p>
```

the `#box` variable is associated with the input element itself
* `box.value` gets the value of the input element
* angular will only update the `box` variable when it evaluates the tag
* evaluating the tag only occurs because `(keyup)` tells it to listen for events
* the 'template statement' `="0"` does nothing, but it's still necessary


## forms
cookbook-style chapter

`[()]` syntax combines `[]` binding and `()` event binding
* `[(ngModel)]` can be split into both parts

```
	<input type="text" class="form-control" id="name"
		   required
		   [ngModel]="model.name" name="name"
		   (ngModelChange)="model.name = $event" >
```

`ngModelChange` is an angular event emitter, not a DOM event
* it returns the value of the input box

## dependency injection
a coding pattern where a class receives dependencies rather than creating them
* if a class depends on an instance of another, it should accept an instance
  in a constructor as a parameter rather than creating an instance itself

angular has its own dependency injection framework
* because why not
* `let car = injector.get(Car);`
* it can actually be used as a standalone product for other projects

configure angular's injector by registering your service providers
* registration can happen either in `NgModule` or in application components

any registered providers are available to that component and all children
angular creates any necessary injectors when it creates components
injectors can be created manually if needed

dependencies are singletons within the scope of an injector
services are reused within the scope for every component that needs it
you can still create two of a service by registering it twice (you shouldn't)

```
	providers: [
		NewLogger,
		{ provide: OldLogger, useClass NewLogger }
	] // this creates two instances of NewLogger

	providers: [
		NewLogger,
		{ provide: OldLogger, useExisting: NewLogger }
	] // uses the existing NewLogger class instead of creating one
```

the `@Injectable()` decorator marks a service available for injection
angular looks for this injectable service in component constructor parameters

the metadata produced by the typescript transpiler is essential to injection
* using any decorator ensures that metadata for that class is available
* there's nothing special about `@Injectable()`(?)
* `@Directive`, `@Pipe`, and `@Component` are subtypes of `@Injectable()`
* `@Injectable()` is rarely needed, but it can make things clearer

can provide an object as a provider directly, instead of a class to instantiate

```
	providers: [
		NewLogger,
		{ provide: OldLogger, useValue: LoggingObject }
	]
```

can use factory methods, spelling out dependencies for the method
this is complicated, but effective
```
	export let heroServiceProvider = {
		provide: HeroService,
		useFactory: heroServiceFactory,
		deps: [Logger, UserService]
	};
```

the tokens that the dependency injector uses are keys to a map of dependencies
when dependencies are class instances, classnames can serve as tokens
when dependencies are not classes, we have to get creative about the tokens used
use `OpaqueToken` to assign a token to a dependency

use `@Optional()` to tell angular a dependency is not required
* angular may set it to null if it wants


## template syntax
`{{ }}` takes template expressions, which are javascript without side-effects
execution context for template expressions is the component

event binding is done with 'template statements'
these statements are almost javascript
context for template statements is also the component

html attributes are different than dom properties
* properties change, attributes don't
angular binds to properties
attributes only serve to initialize values

`<button [disabled]="isUnchanged">disabled</button>`
this binds to the `disabled` property
binding to the attribute would do nothing when it changed
binding to the property changes the state of the button

use `[attr.<attribute>]=<template expression>` to bind to attributes
add and remove css classes with the class binding
`<div [class.special]="isSpecial">adds special class if "isSpecial" is truthy...`
use `NgClass` directive to manage css classes in general cases
use `NgStyle` directive for styles

### event binding
`<button (click)="onSave()">Save</button>`
information about the event is held in `$event`
* contents are determined by the type of the event
* regular dom events make `$event` a regular dom event object

create custom events using `EventEmitter` in a directive (component)

```
	deleteRequest = new EventEmitter<Hero>();
	delete() {
		this.deleteRequest.emit(this.hero);
	}
```

angular provides a lot of directives to help with common tasks
`ngModel` to bind to certain html elements, such as `<input>`
* `<input [(ngModel)]="currenthero.firstName">`
`NgClass` to set multiple classes for an element
* `<div [ngClass]="setClasses()">...</div>`
`NgStyle` to set multiple in-line styles
`ngIf` to conditionally remove things from the dom
`ngSwitch` to c-style switch in tags
`ngFor` to repeat html elements for iterables in the component

questions
=========
what determines component parent-child hierarchy?

topics
======
### overview
* ideal uses
* strengths
* adoption

### how to use it / structure
* modules are the building blocks
 * correspond to features
* components hold logic that drives views
 * regular javascript class, decorated
* templates contain view, binding info
 * tied to components via component metadata
 * most features of template languages are there
  * looping, data binding, conditional display


### behind the scenes
* javascript imports exports vs angular imports exports
* decorators
* method of change detection
* zones

### braintree
* working with braintree
 * basic running
 * binding to valid state

slides
======





