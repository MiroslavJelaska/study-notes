#▶[Dave Smith - Deep Dive into Custom Directives - NG-Conf 2014](https://www.youtube.com/watch?v=UMkd0nYmLzY)  ★★★
##When to use directives?
* If you want a reusable HTML component
<pre><code>&lt;my-widget&gt;</code></pre>
* If you want reusable HTML behavior
<pre><code>&lt;div ng-click="..."&gt;</code></pre>
* If you want to wrap a jQuery plugin
<pre><code>&lt;div ui-date&gt;</code></pre>
*Almost any time you need to interface with the DOM

##Getting Started
1. Create a module for your directives <pre><code>angular.module('MyDirectives', [])</code></pre>
2. Load the module in your app: <pre><code>angular.module('MyApp', ['MyDirectives']);</code></pre>
3. Register your directive:
<pre><code>angular.module('MyDirectives', []).directive('myDirective', function(){ //TODO Make this do something})</code></pre>
4. Use your directive in your HTML

* Why are they in separate module? To make them independatly unit testable. (Testing them without loading a whole app)

##Your First Directive
How about a directive that extends a text <input> to automatically highlight its text when the user focuses it?
<pre><code> &lt;input type="text" select-all-on-focus &gt;</pre></code>
<pre><code>angular.module('MyDirective').
directive('selectAllOnFocus', function(){
  return {
    restrict: 'A',
    link: function(scope, element){
      element.mouseup(function(event){
        event.preventDefault();
      });
      element.focus(function(){
        element.select();
      });
    }
}});
</pre></code>
Note: The link() function arguments are positional, not injectable.

##Restrict
###Attributes only:
<pre><code>
restrict: 'A'
&lt; div my-directive&gt; &lt;/div&gt;
</pre></code>
Whent to use: Behavioral HTML, like ng-click
###Elements only:
restrict: 'E'
&lt;my-directive&gt; &lt;/my-directive&gt;
</pre></code>
Whent to use: Component HTML, like a custom widget
###Weird ones: (rarely used in practice)
'C' for classes and 'M' for comments

You can combine them: 'AE'
##Templates
Two options for specifying the template:
* `template` Define the HTML in JS
* `templateUrl` Use a URL to an HTML partial
Tip: You can avoid the round trip to the server with:
```
<script type="text/ng-template" id="/partials/my-social-buttons.html">
  <div>
    <a href="x"> facebook </a>
    <a href="y"> twitter </a>
  </div>
</script>
```
How Angular downloads templates:
``` $http.get(templateUrl, { cache: $templateCache }) ```

##Interactive widget example
Let's make a custom search box
``` <my-searchbox search-text="theSearchText" placeholder="Search text"></my-searchbox>```
* directive has Isolate scope
```
angular.module('MyDirectives')
.directive('mySearchbox', function(){
  return{
    restrict: 'E',
    scope: {
      searchText: '=',
      placeholder: '@',
      usedLucky: '='
    },
    template:
    '<div>' +
    ' <input type="text" ng-model="tempSearchText" />' +
    ' <button ng-click="searchClicked()"> Search </button>' +
    ' <button ng-click="luckyClicked()"> I\'m feeling lucky </button>' +
    '</div>'
  }
})
```
##Isolate scope
By default, a directive does not have its own scope.(e.g. ng-click)
But you can give your directive its own scope like this:
``` 
scope: {
  someParameter: '='
}
```
Or like this: ``` scope: true``` 
**Important:** It does not inherit from the enclosing scope
###Why can it not inherit from the enclosing scope?
* To keep a clean interface between the directive and the caller
* To prevent the directive from (easily) accessing the caller's scope
* To encourage direcrive reuse
* To discourage guilty knowledge about the directive's environment
* To make the directive testable in isolation
###Scope Parameters
Each scope parameter can be passed through HTML attributes:
```
scope: {
  param1: '=', //two-way binding (reference)
  param2: '@', // one-way expression (top down)
  param3: '&' // one-way behavior (bottom up)
}
```
Examples:
```
<div my-directive
  param1="someVariable"
  param2="My name is {{theName}}, and I am {{theAge + 10}}."
  param3="doSomething(theName)">
```
###attrs : Other way to pass data to directives
The ```attrs``` object gives read/write access to HTML attributes:
```
directive('myDirective', function(){
  return {
    link: function(scope, element, attrs){
      var options = scope.$eval(attrs.myDirective)
    }
  }
})
```
Usage:
```
function MyController($scope){
  $scope.someVariable = 42
}
```
```
<div my-directive="{foo: 1, bar: someVariable}">
```
```options``` will contain this Js object:
```{foo:1, bar: 42}```

Use ```attrs.$set()``` to change HTML attrivute values.
Use ```$attrs.observe()``` to be notified when HTML attrivutes change. (Independant of digest loop, litlle bit faster)

##Compile and Link
**Compile** - Convert an HTML string into an Angular template.
**Link** - Insert an Angular template into the DOM with a scope as context.
###Like in hanglbars
Handlebars uses similar concept to Angular's:
```
var template = Handlebars.compile('<div> ... </div>');
var html = template({ foo: 1, bar: 2});
```
In this contect, ```template``` is a lot like a link function.
**The difference**
* Handlebars' link function returns a string
* Angular's link function creates DOM at a scpecified location
 
###When should I use it?
Directives that use compile: ng-repeat, ng-if, ng-switch
* When you need to add/remove DOM element after link time
* When you need to reuse a template multiple times

###[Lazy loading expensive DOM](https://youtu.be/UMkd0nYmLzY?t=25m54s)
##Transclusion
Think of transcludable directives as a picture frame. Frames html, and alows the caller to specify content.
Tell Angular where to put the caller's content with ng-transclude. Inserts it as a child.
```
directive('myDialog', function(){
  return {
    restriction: 'E',
    transclude: true,
    template: 
      '<div class="modal">' +
      ' <div class="modal-body" ng-transclude></div'+
      '</div'>
  }
})
```
