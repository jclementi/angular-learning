angular class
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


topics
======
* strengths
* ideal uses
* adoption

