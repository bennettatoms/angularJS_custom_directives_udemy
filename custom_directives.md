# Dan Wahlin's course on Building Custom Directives in AngularJS

Topics to be covered: 
  - DDOs: directive definitiion objects
  - The $compile service
  - The link() function
  - compile() and pre vs. post link
  - Transclusion
  - Isolate scope

## The Role of Directives
  Definition: Markers on a DOM element that tell Angular JS's HTML compiler ($compile) to attach a specified behavior to that DOM element

  1) They can manipulate the DOM 
  2) They can iterate through data
  3) They can handle events
  4) They can modify CSS classes
  5) They can validate data
  6) They can do data binding

  Forms
    - ng-maxlength
    - ng-minlength
    - ng-pattern
    - ng-required
    - ng-submit

  Behavior
    - ng-blur
    - ng-change
    - ng-checked
    - ng-click
    - ng-key
    - ng-mouse

  Data Binding
    - ng-bind
    - ng-href
    - ng-init
    - ng-model
    - ng-src
    - ng-style

  DOM
    - ng-disabled
    - ng-cloak
    - ng-hide
    - ng-if
    - ng-repeat
    - ng-show
    - ng-switch
    - ng-view

  Application
    - ng-app
    - ng-controller
  
  3rd Party Directives
    - UI-Bootstrap
    - AngularStrap
    - Angular UI-Grid
    - Angular Translate (for multilingual apps)


## Creating a Hello World Directive


Start with a wrapper function -- pulls everything out of the global scope and keep everything clean (can do for modules, directives, controllers, services, etc.)

helloWorldDirective.js: 

(function() {
  
  var app = angular.module('directivesModule', []);

  app.directive('helloWorld', function() {
    return { // return an object literal, a DDO -- directive definition object
      template: 'Hello World'
    };
  });

})();

<html ng-app="directivesModule">
  <body>
    <hello-world></hello-world>
    <script src="../../scripts/angular.js"></script>
    <script src="helloWorldDirective.js"></script>
  </body>
</html>


## Directive Categories

What is the style of directive I'm writing?
  1) DOM-driven directives -- show or hide something, have something animated, slide in or out, etc. -- all about DOM manipulation

  2) Data-driven directives -- all about data, using other directives and a controller 

  3) Behavior-driven directives -- all about handling DOM events, on click, keystroke, etc., emits or broadcasts an event

Directive Types
  - Attribute directives -- <span my-dir="expression"></span>

  - Element directives -- <my-dir></my-dir> -- literally creating our own tags that can be reused

  - CSS-class directives -- <span class="my-dir: expression"></span>

  - Comment directives -- <!-- directive: my-dir expression -->
    (not very common)


## Directive Building Blocks

Templates (views) and Scope (how we merge in data and bind it)

With a data-driven directive, you may have a controller inside the directive that is used to help create the html that will be outputted

 - $compile provider -- at the bottom of the directive stack -- the engine that revs up and makes directives possible so we can put them in a view and render html

 - DDO -- Directive Definition Object -- next up in the stack -- an object literal that has very specific properties, tells the compile provider what it needs to render -- template, controller, DOM-manipulation code, etc.

 - template

 - scope -- we can inherit scope or create an isolate scope

The $compile provider defined: it compiles an HTML string or DOM into a template and produces a template function which can then be used to link scope and the template together.

The $compile process
  Start with a template, which gets fed into the compile provider via the DDO
  The provider merges in scope via a template function, and then it outputs html.

$compile and the Directive Definition Object (DDO)
  - $compile provider relies on a Directive Definition Object

  Features of the DDO:
    - Defines the template for the directive
    - Can include DOM manipulation code -- might generate new children, for instance
    - Can define a controller for the directive -- a directive can be like a mini-controller and a mini-view
    - Controls the directive's scope
    - Defines how the directive can be used
    - Can embed other directives in a custom directive

Key DDO properties
  1) restrict: tells the $compile provider what directive type(s) to limit it to -- by default it's an attribute or element out of the box, but might want to use as attribute or element (or want anyone else to)

  2) template: string of a template

  3) templateUrl: external html files -- can be combined with angular's template cache (so it can be embedded in a script tag)

  4) scope: same scope as in controllers, can also create isolate scope within directive

  5) controller: can define a controller for given directive

  6) link: function to manipulate DOM

Example:

angular.module('moduleName')
  .directive('myDirective', function() {
    return {
      restrict: 'EA',                     // restrict to element or attribute
                                          // E = element, A = attribute, C =  
                                             class, M = comment
      scope: {                            // isolate scope
        title: '@'                        // @ reads the attribute value, 
                                             = provides two-way binding, 
                                             & works with functions
      },                          
      template: '<div>{{ myVal }}</div>',
      controller: controller,             // embed controller functions
      link: function(scope, element, attrs) { }
    }
  })

We need directives because controllers are not supposed to touch the DOM, and directives fill that gap.
Directives that are data driven are typically controllers and views embedded in a directive.

## Shared and Isolate Scope

Isolate scope allows directives to be more reusable
Three properties:
  - @ local scope property
  - = local scope property
  - & local scope property

### Shared Scope
  Can think of it like bunkbed, parent scope on top, and child scope nested below

  Similar to when you nest controllers in AngularJS

  If you want directive to be reusable, you should create isolate scope, because you don't know what parent scope will be wherever the directive is employed.

### Isolate scope
  Scope has all its own data, you can get to it, but nothing else can get to it. But they can pass you things, communicate using events or pass functions around.
  If you want to pass proerty from parent scope into isolate scope, there is a way to do that.

How to get data from parent scope into child scope, and how to pass data out to parent. There's a wall between the scopes, but we can pass data back and forth using pipes in the wall.

If controller has $scope.customers = [], if there's shared scope (and there is naturally shared scope if directive has parent scope in controller), the directive can do whatever it wants with the $scope.customers array.

Example of shared scope:

  var app = angular.module('myModule', []);

  app.controller('Controller', ['$scope', function($scope) {
    $scope.customer = {
      name: 'David',
      street: '1234 Anywhere St'
    };
  }]);

Shared Scope in Directive:
  app.directive('sharedScope', function() {
    return {
      template: 'Name: {{customer.name}}, Street: {{customer.street}}'
    }
  });

  <div shared-scope></div>

Isolate Scope in Directive:
  app.directive('isolateScope', function() {
    return {
      scope: {},
      template: 'Name: {{customer.name}}, Street: ' + {{customer.street}}'
    }
  });

  <div isolate-scope></div> // no data will display, because we have no customer


If you create isolate scope within the directive, the directive has no knowledge of the $scope.customers array. 
We have to use the local scope properties to pass data through the wall.

directivesController.js:

var app = angular.module('directivesModule', []);

app.controller('CustomersController', ['$scope', function ($scope) {
  var counter = 0;
  $scope.customer = {
    name: 'David',
    street: '1234 Anywhere St.'
  };
  
  $scope.customers = [
    {
      name: 'David',
      street: '1234 Anywhere St.'
    },
    {
      name: 'Tina',
      street: '1800 Crest St.'
    },
    {
      name: 'Michelle',
      street: '890 Main St.'
    }
  ];

  $scope.addCustomer = function () {
    counter++;
    $scope.customers.push({
      name: 'New Customer' + counter,
      street: counter + ' Cedar Point St.'
    });
  };

  $scope.changeData = function () {
    counter++;
    $scope.customer = {
      name: 'James',
      street: counter + ' Cedar Point St.'
    };
  };
}])

.directive('sharedScope', function () {
  return {
    template: 'Name: {{customer.name}}<br /> Street: {{customer.street}}'
  };
});

.directive('isolateScope', function () {
  return {
    scope: {},
    template: 'Name: {{customer.name}}<br /> Street: {{customer.street}}'
  };
});

<html ng-app="directivesModule">
<head>
  <title></title>
</head>
  <body ng-controller="CustomersController">
    <h2>Shared Scope Directive</h2>
    <shared-scope></shared-scope>
    <br>
    <h2>Isolate Scope Directive</h2>
    <isolate-scope></isolate-scope>
    <br>
    <script src="../../scripts/angular.js"></script>
    <script src="../../scripts/directivesController.js"></script>
    <script src="sharedScopeDirective.js"></script>
    <script src="isolateScopeDirective.js"></script>
  </body>
</html>

## Local Scope Property

### 1) @ - represents a **string value** that we want to be able to access within the directive
    - a string with one-way binding
    - if we change the value in the directive, it doesn't get passed back as the changed value

    $scope.first = 'Dave';

    in directive:
    scope: { name: '@' } // you can pass a string value into this scope
                         // think of anything within the curly brackets as
                         // literally a property of the directive

    <my-directive name="{{first}}"></my-directive>

    Because the value of 'first' is a string value, we just do a data-binding expression on the name attribute in the directive.
    At runtime, will pass in 'Dave' and the name in the directive will resolve to 'Dave'

    Since we created the 'name' property -- the pipeline that goes through the wall between parent and isolate directive scope -- we know we can access that property in the html directive.

    .directive('isolateScopeWithString', function () {
      return {
        scope: {
          name: '@'
        },
        template: 'Name: {{customer.name}}<br /> Street: {{customer.street}}'
      };
    });

    <isolate-scope-with-string name="{{customer.name}}"></isolate-scope-with-string>

    We have to put the customer.name in curly brackets because we want the property value rather than the literal string, otherwise will expect the string 'customer.name' 

    If for some reason you don't want the user to pass in just a name, but rather the full name, could put scope: { name: '@fullname' } instead. 
    Can have it named differently externally than internally, but can get convoluted.


### 2) = - two-way binding, values can be passed in and changed within the directive, and it will also change the property value in the parent scope
  The '=' allows objects or strings or whatever to be passed through, as opposed to the '@', which only allows strings.

  In the html directive, will not use {{}} like you would with '@'. 

  .directive('isolateScopeWithObject', function () {
    return {
      scope: {
        datasource: '='
      },
      template: 'Name: {{datasource.name}} Street: {{datasource.street}}' +
                '<br /><button ng-click="datasource.name=\'Fred\'">' + 
                'Change</button>'
    };
  });

  <isolate-scope-with-object datasource="customer"></isolate-scope-with-object>

Thresholds will need to be configured with =, so that we can set it in the controller, but the user can adjust it as needed

### 3) & - allows parent scope to pass in function to be stored in our directive's isolate scope, and we can then invoke that function (callback) in the parent scope 

  parent controller: 
    $scope.click = function() {};

  directive:
    scope: {
      action: '&'
    }  

  Can invoke external function via directive.
  <my-directive action="click()" />

  .directive('isolateScopeWithEvent', function () {
    return {
      scope: {
        datasource: '=',
        action = '&'
      },
      template: 'Name: {{datasource.name}} Street: {{datasource.street}}' +
                '<br /><button ng-click="action()">Change</button>'
    };
  });

  <isolate-scope-with-event datasource="customer" action="changeData()"></isolate-scope-with-event>

Now the changeData() function from the controller above will be stored in the directive via the &. 
Can think of 'action' in the directive as an alias.

## The link() Function

Used for DOM manipulation

(This module will focus mostly on DOM-driven directives, with a little bit pf behavior-driven directives)

For example, with calendar directives, would need to use a lot of divs with different CSS layouts. 

Or might want to do something when user hovers over a div, so need to use events.

Both of the above are good use cases for the link() function.

Dan Wahlin prefers to structure his directives, controllers, services, etc. as follows:

(function() {                 // structure directives this way to prevent 
                              // global variables and to just focus
  var linkDemo = function() { // on the function for the directive
    return {

      link: function (scope, element, attrs) { }
      // links the template for the directive to the scope -- or links the 
      // data to the view 

      // 'scope' will be isolate or shared scope --
      // scope is the same object as $scope, but it works differently than 
      // with dependency injection in controllers, etc. -- it will always be 
      // one of the parameters of the link function'

      // 'element' will represent either the element that the directive is 
      // applied to, if it is an attribute, or it may apply to the tag itself,
      // if you define the directive as an element

      // 'attrs' will be all the attributes on that particular element, so 
      // that allows us to get to everything we need, and now we can start to 
      // manipulate things, add children, read values from attributes, etc.

    }
  }

  angular.module('moduleName')
    .directive('linkDemo', linkDemo); // then we find the module, name the 
                                      // directive, and plug the function in
}());

Angular uses jqLite for DOM manipulation -- jqLite us a tiny, API-compatible subset of jQuery [source](https://docs.angularjs.org/api/ng/function/angular.element)

Can also use vanilla javaScript.

If you load jQuery before Angular in your app, then Angular will use jQuery instead of jqLite -- not typically necessary

Key jqLite functions (don't require the $ sign):
** angular.element() ** to get to the actual element we want to work with
** addClass() ** to add CSS class (or css() if you just want to use inline style)
** attr() ** to get to attributes and manipulate those
** on() / off() ** for different types of events (can also use bind() and
   unbind(), but on and off are more common in the jQuery world)
** find() ** only finds tags -- in jQuery can find by class or id or complex types of selectors
** append() / remove() ** to add or remove child nodes
** html() / text() ** to alter inner html or inner text on a given tag
** ready() ** to wait until DOM is ready

[jqLite documentation](https://docs.angularjs.org/api/ng/function/angular.element)

Example: DOM/behavioral-driven
(function() {        

  var linkDemo = function() { 
    return {
      restrict: 'A', // attribute only, don't want to use it as a tag, because
                     // we'll want to be able to attach this directive to 
                     // anything -- handle different types of events
      link: function (scope, elem, attrs) { 
        elem.on('click', function() {
          elem.html('you clicked me'); // will change the text displayed from
                                       // 'Click Me!' to 'you clicked me'
        });
        elem.on('mouseenter', function() {
          elem.css('background-color', 'yellow'); // could also use addClass()
        });
        elem.on('mouseleave', function() {
          elem.css('background-color', 'white');
        });
      }
    }
  }

  angular.module('directivesModule', [])
    .directive('linkDemo', linkDemo); 

}());

to use the above directive:
<div link-demo>Click Me!</div>


### Building a TableHelper Directive -- a directive that takes any kind of data and maps it out to a table


our tableHelper.html:
<!DOCTYPE html>
<html>
<head>
  <title>Tabke Helper DOM Directive</title>
  <style>
    h3 { cursor: pointer; }
    th { text-align: left; background-color: #ccc; cursor: pointer; }
    td { width: 150px; }
    .tableHelper { font-family: 'Arial'; }
    .rowCount { font-weight: bold; }
  </style>
</head>
<body>
  <div data-ng-app="directivesModule"
       data-ng-controller="CustomersController">
    <h3 ng-click="addCustomer()">Table Helper DOM Directive</h3>
    <table-helper 
      datasource="customers"
      columnmap="[{name: 'Name'}, {street: 'Street'}, {age: 'Age'}, {url: 'URL', hidden: true}]"> 
    </table-helper> 
  </div>
  <script src="../../scripts/angular.js"></script>
  <script src="../../scripts/directivesController.js"></script>
  <script src="tableHelper.js"></script>
</body>
</html>

our directive, tableHelper.js:
(function() {        

  var tableHelper = function() { 
    var template = '<div class="tableHelper"></div>',

        link = function(scope, element, attrs) {
          var headerCols   = [],
              tableStart   = '<table>',
              tableEnd     = '</table>',
              table        = '',
              visibleProps = [],
              sortCol      = null, 
              sortDir      = 1;

          // we need to know if datasource property changes at all, and if it
          // does, we'll need to re-render the table
          // we'll use $watchCollection since datasource is an array

          scope.$watchCollection('datasource', render); 
            // if changes, call render function

          wireEvents();

          function render() {
            if (scope.datasource && scope.datasource.length) {
              table += tableStart;
              table += renderHeader();
              table += renderRows() + tableEnd;
              renderTable();
            }
          }

          // on a click of the actual element that represents this directive 
          // (since this is restricted to 'E')

          function wireEvents() {
            element.on('click', function(event) {
              // if the element that was clicked was a table header
              if (event.srcElement.nodeName === 'TH') {
                // grab the name of the column
                var val = event.srcElement.innerHTML;

                // use the columnmap to control what displays
                // if we have a columnmap, we'll use the mapped value for the 
                // column name and convert it back to the property name, 
                // otherwise we'll just use the property name
                var col = (scope.columnmap) ? getRawColumnName(val) : val;

                // now we'll sort by the property name
                if (col) sort(col);
              }
            });
          }

          function sort() {
            // See if they clicked on the same header
            // If they did, reverse the sort
            if (sortCol === col) sortDir = sortDir * -1;

            sortCol = col;
            scope.datasource.sort(function(a, b) {
              if (a[col] > b[col]) return 1 * sortDir;
              if (a[col] < b[col]) return -1 * sortDir;
              // if they're equal, return 0
              return 0;
            })
            render();
          }

          function renderHeader() {
            var tr = '<tr>';

            // iterate through all the properties and get the mapped version
            for (var prop in scope.datasource[0]) {
              var val = getColumnName(prop);
              if (val) {
                // Track visible properties to make it fast to check them later
                visibleProps.push(prop);

                // create a table header with each mapped column name
                tr += '<th>' + val + '</th>';
              }
            }
            tr += '</tr>';

            // put the column headers inside the thead tags and return the 
            // entire thead string
            tr = '<thead>' + tr + '</thead>';
            return tr;
          }

          function renderRows() {
            var rows = '';

            // loop through all the data items in the datasource property
            for (var i = 0, len = scope.datasource.length; i < len; i++) {
              rows += '<tr>';

              // add a new row for every data item
              var row = scope.datasource[i];

              // iterate through each property in the data item and add that
              // property's value to a new cell in the row, as long as the
              // property is not hidden (i.e. a url)
              for (var prop in row) {
                if (visibleProps.indexOf(prop) > -1) {
                  rows += '<td>' + row[prop] + '</td>';
                }
              }
              rows += '</tr>';
            }

            // place all the rows inside the tbody tags and return the entire 
            // tbody string
            rows = '<tbody>' + rows + '</tbody>';
            return rows;
          }

          function renderTable() {
            // add a little info at the bottom of the table about the number
            // of rows
            table += '<br /><div class="rowCount">' + scope.datasource.length + ' rows</div>';

            // update the template with the actual data
            // recall the template from above -- '<div class="tableHelper">
            // </div>'

            // this is all the HTML that Angular will create when it 
            // encounters the <table-helper> tag 

            element.html(table);
            table = ''; 
          }

          function getRawColumnName(fiendltCol) {
            var rawCol;
            scope.columnmap.forEach(function(colMap) {
              for (var prop in colMap) {
                if (colMap[prop] === friendlyCol) {
                  rawCol = prop;
                  break;
                }
              }
              return null;
            });
          }

          function filterColumnMap(prop) {
            var val = scope.columnmap.filter(function(map) {
              if (map[prop]) {
                return true;
              }
              return false;
            });
            return val;
          }

          function getColumnName(prop) {
            if (!scope.columnmap) return prop;
            var val = filterColumnMap(prop);
            if (val && val.length && !val[0].hidden) return val[0][prop];
            else return null;
          }

        };

    return {
      restrict: 'E', // element only
      scope: {
        columnmap: '=', // bi-directional -- even though we will not be
                        // updating the data, the value passed in will be an 
                        // object, so cannot use @, which only works with 
                        // strings -- will allow us to map properties to 
                        // column names

        datasource: '=' // also bi-directional
      },
      link: link,
      template: template
    };
  };

  angular.module('directivesModule', [])
    .directive('tableHelper', tableHelper); 

}());


### Requiring ngModel -- might want to use built-in AngularJS properties in custom directives, by using require

Use the 'require' statement inside the DDO to include Angular's built-in (or your own custom) directives inside the custom directive you are writing.

We'll create a new table helper directive, but this time we'll use ng-model:
our directive, tableHelperWithNgModel.js:
(function() {        

  var tableHelperWithNgModel = function() { 
    // ...template and link function will stay the same, but instead of using 
    // datasource, we'll use ng-model inside the directive
    // [ 
    //   <table-helper-with-ng-model 
    //     ng-model="customers"
    //     columnmap="[{name: 'Name'}, {street: 'Street'}, {age: 'Age'}, {url: // 'URL', hidden: true}]"> 
    //   </table-helper-with-ng-model> 
    // ]

    return {
      restrict: 'E',
      require: 'ngModel', // ng-model will be imported into our directive at
                          // runtime **NOTE:** if the directive is missing the 
                          // ng-model attribute, it will throw an error -- can
                          // evade having to use ng-model by putting "require:
                          // '?ngModel'" instead

                          // also, you can allow ng-model to be present either
                          // on the element or on a parent element like so:
                          // "require: '^ngModel'" (can also make optional
                          // with '?^ngModel')

                          // if you only want to search for the ng-model
                          // attribute at the parent level:
                          // "require: '^^ngModel'"
      scope: {
        columnmap: '='
      },
      link: link,
      template: template
    };
  };

  angular.module('directivesModule')
    .directive('tableHelperWithNgModel', tableHelperWithNgModel); 

}());

Now that we have ng-model required, we can add ngModel to the parameters of the link function, and it will represent the [ngModelController](https://docs.angularjs.org/api/ng/type/ngModel.ngModelController)

The ngModelController has a bunch of different methods (as you can see in the docs) like $render(), $isEmpty(value), $setValidity(validationErrorKey, isValid), etc., so adding the ngModel param to the link function, we get access to all of the controller's methods.

So now we need to know when the ngModel changes.

Inside the link function, we'll have to adjust the render function to watch for changes to the ngModel --> we'll look at four ways to accomplish this:

  1) **$observe(value, callback);** -- Dan Wahlin recommends to use $observe 
     when you're only watching attributes for changes, and the syntax would be:
     attrs.$observe(value, function(value) { render(); // or do whatever } )

  2) **scope.$watch(value, render);**

  3) **scope.$watch(function that returns value to watch, function to exec 
     when value changes)**

  4) **$render()** -- doesn't do a deep watch, but watches for changes in 
     $modelValue and $viewValue and is very efficient

  // Watch for ngModel to change. Required since the $modelValue will be 
  // NaN initially
  
  attrs.$observe('ngModel', function(value) {
    // the value of ngModel is what we're interested, so we'll watch 
    // for a newValue
    scope.$watch(value, function(newValue) {
      render();
    });
  });

  // $watch the actual value, then execute render() when changes

  scope.$watch(attrs.ngModel, render);

  // give $watch a function rather than the value directly -- a little more 
  // efficient than the earlier scope.$watch

  scope.$watch(function() {
    // return the value that you want to watch
    return ngModel.$modelValue;
  }, function(newValue) { // when the value changes, get the new val and render
    render();
  });

  // use the render() method in the NgModelController -- only gets invoked when $modelValue and/or $viewValue change

  ngModel.$render = function() {
    render();
  };

  // update the render function for ngModel -- he's added the var datasource
  // to the top of the link function, so it's accessible here
  function render() {
    if (ngModel && ngModel.$modelValue.length) {
      datasource = ngModel.$modelValue;
      table += tableStart;
      table += renderHeader();
      table += renderRows() + tableEnd;
      renderTable();
    }
  }


### Using $parse and $eval -- might need to pass objects into directives as strings 
We're back to our tableHelper with datasource example.

Remove columnmap from scope object in DDO, and now when we assign the value to the columnmap attribute in the html directive ("[{name: 'Name'}, {street: 'Street'}, {age: 'Age'}, {url: 'URL', hidden: true}]"), Angular just sees it as a string and will process is at such.

We'll have to use $parse or $eval to have Angular convert the string into the array of objects that it's meant to be.

He has added var columnmap = null to the top of the link function.

Then, after: 

  scope.$watchCollection('datasource', render); 
    // if changes, call render function

We'll add:

  columnmap = scope.$eval(attrs.columnmap); 
    // $eval will automatically evaluate the string and convert it -- uses the
    // $parse service under the hood

OR, could do the following:
  
  // have to inject $parse dependency into the directive declaration... 
  // var tableHelper = ['$parse', function($parse) { 

  // columnmap = $parse(attrs.columnmap); ** DID NOT WORK, see note below**
    // $parse returns a function, so in the above columnmap is a function, so
    // you actually have to invoke the function in order to get the output of
    // the function, the correct object

  columnmap = $parse(attrs.columnmap)();


### Building a Google Maps Directive
We'll build a link function that will work with Google maps's geolocation

<!DOCTYPE html>
<html>
<head>
  <title>Geolocation Mapping Directive</title>
</head>
<body>
  <h3>Geolocation Mapping Directive</h3>
  <map-geo-location height="400" width="600"></map-geo-location>
  <br>
  <script src="../../scripts/angular.js"></script>
  <script src="http://maps.google.com/maps/api/js?sensor=false"></script>
  <script src="mapGeolocation.js"></script>
</body>
</html>

We're going to create a directive where all you need to do is give the directive a height and a width, and the directive will render a google map with your location plotted on it.

mapGeolocation.js:

(function() {
  
  var mapGeolocation = ['$window', function($window) {
    var template = '<p><span id="status">looking up geolocation...</span></p>' 
                 + '<br><div id="map"></div>',

        mapContainer = null, // represents the map div in template
        status = null;

    function link(scope, elem, attrs) {
      // we'll use jqLite functionality to find the elements for the status 
      // span and the map div and assign them to variables

      status = angular.element(document.getElementById('status'));
      mapContainer = angular.element(document.getElementById('map'));
        // by wrapping the vanilla javaScript document.getElementById functions
        // in angular.element, it gives the status and mapContainer variables
        // jqLite functionality, so we can then use jqLite methods like text(),
        // append(), html(), and so forth.

      mapContainer.attr('style', 'height:' + scope.height + 
                        'px;width:' + scope.width + 'px');

      // lastly, we'll use the $window object to give us access to html5's
      // navigator object, which has a geolocation API -- has a 
      // getCurrentPosition method that needs success and error callbacks
      $window.navigator.geolocation.getCurrentPosition(mapLocation, geoError);
        // we'll need to write the mapLocation and geoError functions below
    }

    function mapLocation(pos) {
      // use jqLite html() function to update the html 
      status.html('Found your location! Longitude: ' + pos.coords.longitude + 
                  ' Latitude: ' + pos.coords.latitude);

      var latlng = new google.maps.LatLng(pos.coords.longitude, 
                                          pos.coords.latitude);
      
      var options = {
        zoom: 15,
        center: latlng,
        mapTypeControl: true,
        mapTypeId: google.maps.MapTypeIde.ROADMAP
      };

      // the mapContainer is wrapped in the jqLite functionality, but we want
      // the raw DOM element, which is the first object in the jqLite array 
      // object, so either mapContainer[0] or mapContainer.get(0)
      var map = google.maps.Map(mapContainer[0], options);
    
      var marker = new google.maps.Marker({
        position: latlng,
        map: map,
        title: 'Your location'
      });
    }

    // if the html5 geolocation API passes an error object, we'll take it and
    // print its message property
    function geoError(error) {
      status.html('failed lookup: ' + error.message);
    }
  
    return {
      scope: {
        height: '@',
        width: '@'
      },
      link: link,
      template: template
    }
  }];

  angular.module('directivesModule', [])
    .directive('mapGeolocation', mapGeolocation);
}());
  

### Using $compile and $interpolate
The $compile service is used behind the scenes, and it takes in the DDO and builds out the directive.
Additionally, there is a compile function (and also a compile property in the DDO), which returns an object literal, and which you can connect to the $compile service.

There is also an $interpolate service, which converts data-bound objects in {{}} into objects that we can use.

The compile function is repsonsible for loading up the directive template into the DOM one time (and linking it to the scope), and then it can stamp out different instances of the template when they're needed.

Here's the process:
  1) the compile function accesses the raw template and builds it out and stores it before adding it to the DOM -- caches the template

  2) pre-link -- just before the link function is called to bind data to the scope (and this is very rarely used) -- if a directive has nested directives in it, this is where they are called

  3) post-link -- this is actually the link function 

All three of the above are stored in an object literal, and this is actually what the compile function returns.

We'll create a directive that delays creating bindings until the user interacts with them. 
If we have a page with a large number of list items in an ng-repeat, then it takes a lot of memory to generate them and add/watch bindings to them on page load.

<!DOCTYPE html>
<html>
<head>
  <title>Geolocation Mapping Directive</title>
</head>
<body>
  <div ng-controller="CustomersController">
    <h3>Delay Bind Directive</h3>
    The delay-bind directive will cause attribute values to be loaded when a specified event (such as mouseenter) fires.
    <div>
      <ul>
        <li ng-repeat="cust in customers"
            delay-bind="{{::cust.street}}"
            attribute="title"
            trigger="mouseenter">
            <a delay-bind="{{::cust.url}}"
               attribute="href"
               trigger="mouseenter">{{cust.name}}
            </a>
        </li>
      </ul>
      <div>Total Customers: {{::customers.length}}</div>
    </div>
  </div>
  <script src="../../scripts/angular.js"></script>
  <script src="../../scripts/directivesController.js"></script>
  <script src="delayBindWithCompile.js"></script>
</body>
</html>

The <li> element's title attribute will populate with the customer's 'street' value when the user enters with the mouse cursor.

Similarly, the href attribute in the <a> tag will not update with the customer's url until the user mouses over that anchor.

delayBindWithCompile.js:

(function() {

  // inject the $interpolate service, which will be responsible for parsing the
  // bindings, delay-bind="{{::cust.street}}" and delay-bind="{{::cust.url}}"

  var delayBindWithCompile = ['$interpolate', function($interpolate) {
  
    // we're going to be dealing with ng-repeat that creates an <li> and an 
    // <a> element for each item in the customers array -- if there is a large
    // number of them, we don't want to have to parse the template over and
    // over again -- rather pre-process it one time, cache it, and then stamp
    // out an instance for each customer in the array as we loop through

    // the <li> template is everything between the opening and closing tags,
    // including the anchor element, the <a> template is just between and
    // opening and closing anchor tags

    // compile function takes two parameters that represent the template
    // element and the template attributes on that element

    var compile = function(tElem, tAttrs) {
      console.log('In compile');

      // grab the delay-bind attribute values  -- 
      // {{::cust.street}} and {{::cust.url}} -- and convert them
      // to JavaScript functions (that's actually what's happening under the 
      // hood), which we'll store in the interpolateFunc variable

      var interpolateFunc = $interpolate(tAttrs.delayBind);

      // go into the attributes again and set them to null -- 
      // clears them out so that no bindings occur

      tAttrs.$set('delayBind', null);

      // whenever you employ a custom compile function, you need to, at the
      // very least, include a post function in the returned object literal,
      // because that's actually the link function that you would ordinarily
      // have to include in the absence of a compile function

      return {
        pre: function(scope, elem, attrs) {
          console.log('In delayBind pre ' + elem[0].tagName);
        },
        post: function(scope, elem, attrs) {
          console.log('In delayBind post ' + elem[0].tagName);

          // on the event that we've defined in our element's trigger attribute

          elem.on(attrs.trigger, function(event) {

            // store that attribute ('title' or 'href') in the attr variable

            var attr = attrs.attribute;

            // and run the function we stored in the interpolateFunc 
            // variable on scope and store the result as the val variable 

            var val = interpolateFunc(scope);

            // if there is an 'attribute' attribute on the element, it's value
            // is stored in attr AND if there isn't already an attribute on
            // that element with the same value (i.e. the user hasn't already
            // triggered the binding event)

            if (attr && !elem.attr(attr)) {

              // set attr's value to val, i.e. 'title' to the customer's
              // street and the 'href' to the customer's url

              elem.attr(attr, val);
            }
          });
        }
      };
    };

    return {
      restrict: 'A',
      compile: compile
    };
  }];
  
  angular.module('directivesModule')
    .directive('delayBindWithCompile', delayBindWithCompile);
}());


### Is link() Always Appropriate? 
  There are some other options to use besides link()

<!DOCTYPE html>
<html>
<head>
  <title>Use Link Or Not?</title>
</head>
<body>
  <div ng-app="directivesModule"
       ng-controller="CustomersController">
    Parent Scope Controllers:
    <br><br>
    <ul>
      <li ng-repeat="cust in customers">{{ cust.name }}</li>
    </ul> 
    <h3>Should link() be used?</h3>
    <use-link-or-not datasource="customers" add="addCustomer"></use-link-or-not>
  </div>
  <script src="../../scripts/angular.js"></script>
  <script src="../../scripts/directivesController.js"></script>
  <script src="useLinkOrNot.js"></script>
</body>
</html>

useLinkOrNot.js:

(function() {
  var useLinkOrNot = function() {
    var template = '<button id="addItem">AddItem</button><div></div>';

    var link = function(scope, element, attrs) {

      // Create a copy of the original data that's passed in
      var items = angular.copy(scope.datasource),
          button = angular.element(document.getElementById('addItem'));

      button.on('click', addItem);
      render();

      function addItem() {
        var name = 'New Directive Customer';

        // Call external function passed in with '&'
        scope.$apply(function() {
          scope.add()(name);
        });

        // Add new customer to the local collection
        items.push({
          name: name
        });

        render();
      }

      function render() {
        var html = '<ul>';

        // iterate through the items array and add each item's name to the list
        for (var i=0, len = items.length; i < len; i++) {
          html += '<li>' + items[i].name + '</li>';
        }
        html += '</ul>';

        // plug the full list of items into the div
        element.find('div').html(html);
      }
    }

    return {
      restrict: 'EA',
      scope: {
        datasource: '=',
        add: '&'
      }
      link: link,
      template: template
    }
  }

  angular.module('directivesModule')
    .directive('useLinkOrNot', useLinkOrNot);
}());

The issue with using a link function like the above is that it basically just does what an ng-repeat directive does, although this version is a little more performant because we're bypassing some of the wrappers that something like an ng-repeat entails, but it's less maintainable code.


## Using Controllers

Can also leverage controllers inside of directives

### Using a Controller in a Directive

Recall the three catgories for Angular directives:
  1) DOM-driven
  2) Behavior-drivem
  **3) Data-driven** - this is where we'll employ controllers inside directives

How to add controller to a directive? Add DDO properties.

angular.module('directivesModule')
  .directive('withController', function() {
    return {
      restrict: 'EA',
      scope: {},
      **controller: function($scope) {},**
      template: '<div></div>'
    };
  });

With a link function, you have to write every bit of code that touches the DOM and manipulates it, handles events, updates the DOM, which can require using a lot of jqLite or vanilla JavaScript.

With a controller, we can just create a baby view and leverage the existing Angular concepts/direcitves we already know to manipulate the DOM.

Can also use controllerAs (and starting with 1.3, have to also use bindToController: true):

angular.module('directivesModule')
  .directive('withController', function() {
    return {
      restrict: 'EA',
      scope: {},
      controller: function() {},
      **controllerAs: 'vm',
      bindToController: true,**
      template: '<div></div>'
    };
  });


### Replacing Link() with a Controller

<!doctype html>
<html ng-app="directivesModule">
  <body>
    <div ng-app="directivesModule"
         ng-controller="CustomersController">
      <h3>Without Controller</h3>
      <without-Controller datasource="customers" add="addCustomer">
      </without-controller>
      <br>
      <h3>With Controller</h3>
      <with-controller datasource="customers" add="addCustomer">
      </with-controller>
      <br>
    </div>
    <script src="../../scripts/angular.js"></script>
    <script src="../../scripts/directivesController.js"></script>
    <script src="withoutController.js"></script>
    <script src="withController.js"></script>
  </body>
</html>

withController.js:
(function() {
  var withController = function () {

    // we'll create variables that represent our view and our controller
    var template = '<button ng-click="addItem()">Add Item</button><ul>' +
                   '<li ng-repeat="item in items">{{ ::item.name }}</li></ul>',

        // injecting $scope will give us access to any of the isolate scope
        // properties included below

        controller = ['$scope', function($scope) {

          init();

          function init() {

            // first thing we do is copy the datasource object so we don't 
            // alter the original data, just as before...
      
            // since datasource is locally scoped via '=', any changes we make
            // to it locally will update the outer value...

            // we'll iterate through the array of copied items

            $scope.items = angular.copy($scope.datasource);
          }

          $scope.addItem() {
            var name = 'New Directive Controller Item';

            // why this approach?

            $scope.add()(name);

            $scope.items.push({
              name: name
            });
          }
        }];


    return {
      restrict: 'EA', //Default in 1.3+
      scope: {
          datasource: '=',
          add: '&',
      },
      controller: controller, // define controller just as we did link in 
                              // prev examples
      template: template
    };
  };

  angular.module('directivesModule')
    .directive('withController', withController);
}());


### Using controllerAs

<!DOCTYPE html>
<html>
<head>
  <title>Shared Scope Directive (controllerAs)</title>
</head>
<body>
  <div data-ng-app="directivesModule"
       data-ng-controller="CustomersController">
    <br>
    <h3>Using controllerAs</h3>
    <br>
    <controller-as datasource="customers" add="addCustomer">
    </controller-as>
    <br>
  </div>
  <script src="../../scripts/angular.js"></script>
  <script src="../../scripts/directivesController.js"></script>
  <script src="controllerAs.js"></script>
</body>
</html>

controllerAs.js:
(function() {

  var controllerAs = function () {

    // have to adjust template to account for vm rather than $scope
    var template = '<button ng-click="vm.addItem()">Add Item</button><ul>' +
                   '<li ng-repeat="item in vm.items">{{ ::item.name }}</li></ul>',

        // with controllerAs, remove $scope dependency and replace with vm
        controller = function() {

          var vm = this;

          init();

          function init() {
            vm.items = angular.copy(vm.datasource);
          }

          vm.addItem = function() {
            var name = 'New Directive Controller Item';
            vm.add()(name);
            vm.items.push({
              name: name
            });
          }
        };

    return {
      restrict: 'EA', //Default in angular ^1.3
      scope: {
          datasource: '=',
          add: '&',
      },
      controller: controller,
      controllerAs: 'vm', // ADDED
      bindToController: true, //ADDED (required for angular ^1.3)
      template: template
    };
  };

  angular.module('directivesModule')
    .directive('controllerAs', controllerAs);
}());

### Adding a Controller to TableHelper

<!DOCTYPE html>
<html>
<head>
  <title>Table Helper DOM Directive</title>
  <style>
    th { text-align:left; background-color: #ccc;cursor:pointer;}
    td { width:150px; }
    .tableHelper { font-family: 'Arial' }
    .rowCount { font-weight: bold; }
  </style>
</head>
<body>
  <div data-ng-app="directivesModule"
       data-ng-controller="CustomersController">
    <br>
    <h3 ng-click="addCustomer()">Table Helper Controller Directive</h3>
    <table-helper datasource="customers"
      columnmap="[{name: 'Name'}, {street: 'Street'}, {age: 'Age'}, {url: 'URL', hidden: true}]">
    </table-helper>
  </div>
  <script src="../../scripts/angular.js"></script>
  <script src="../../scripts/directivesController.js"></script>
  <script src="tableHelper.js"></script>
</body>
</html>

tableHelper.js:
(function() {

  var tableHelper = function () {

    var controller = ['$scope', function($scope) {
      var vm = this,
          visibleProps = [];

      vm.columns = [];
      vm.reverse = false;
      vm.orderby;

      $scope.$watchCollection('datasource', getColumns);

      //Handle sorting of data as user clicks on a column in the table
      
      vm.sort = function(col) {
        if (vm.columnmap) {

          // The col value is the "friendly" value so we need to figure out
          // the "raw" value

          rawCol = getRawColumnName(col);
          vm.reverse = (vm.orderby === rawCol) ? !vm.reverse: false;
          vm.orderby = rawCol;
        }
        else {
          vm.reverse = (vm.orderby === col) ? !vm.reverse: false;
          vm.orderby = col;
        }
      }

      //Get the columns to display in the table header

      function getColumns() {

        // Since columnmap is an '@' property in this example (to demonstrate
        // we can do that) we need to convert it to an object.
        // Can use $scope.$eval or $parse service ($parse is in another 
        // example)
        // See https://docs.angularjs.org/guide/expression

        vm.columnmap = $scope.$eval(vm.columnmap);

        if (vm.columnmap) {

          //Use columnmap to handle displaying columns

          vm.columnmap.forEach(function(map) {
            if (!map.hidden) {
              for (var prop in map) {
                if (prop !== 'hidden') pushColumns(prop, map[prop]);
              }
            }
          });
        }
        else {

          //No column map so default to raw properties

          for (var prop in vm.datasource[0]) {
            pushColumns(prop, prop);
          }
        }
      }

      // Load each row's values in the proper order based upon order of the 
      // columnmap.

      // We could use ng-repeat="(key,value) in row" on the <td> but the order
      // that data is written out won't match with the headers. This function
      // handles that issue.

      vm.getRowValues = function(row) {
        var sortedValues = [];
        if (vm.columnmap) {
          visibleProps.forEach(function(prop) {
            sortedValues.push(row[prop]);
          });
        }
        return sortedValues;
      };

      function getRawColumnName(friendlyCol) {
        var rawCol;
        vm.columnmap.forEach(function(colMap) {
          for (var prop in colMap) {
            if (colMap[prop] === friendlyCol) {
              rawCol = prop;
              break;
            }
          }
          return null;
        });
        return rawCol;
      }

      function pushColumns(rawCol, renamedCol) {
        visibleProps.push(rawCol);
        vm.columns.push(renamedCol);
      }

    }];

    return {
      restrict: 'E',
      replace: true,
      scope: {
        columnmap: '@',
        datasource: '='
      },
      controller: controller,
      controllerAs: 'vm',
      bindToController: true,
      templateUrl: 'tableHelperTemplate.html' // ADDED -- store template
                                              // externally (see below)
    };
  };

  angular.module('directivesModule')
    .directive('tableHelper', tableHelper);
}());

tableHelperTemplate.html:
<div>
  <table>
    <thead>
      <tr>
        <th ng-repeat="col in vm.columns"
            ng-click="vm.sort(col)">{{col}}</th>
      </tr>
    </thead>
    <tbody>
      <tr ng-repeat="row in vm.datasource | orderBy:vm.orderby:vm.reverse">
        <td ng-repeat="value in vm.getRowValues(row)">{{value}}</td>
      </tr>
    </tbody>
  </table>
  <br />
  <div class="rowCount">{{vm.datasource.length}} rows</div>
</div>

We can do the same exact thing with a controller as we can with a link function.
There can be performance gains to be realized by controlling everything in the link function.
For instance, if you're employing a table that has a lot of data, you might want to have more control over everything and employ a link function.

If the table only has a few hundred rows or fewer, there may only be a minimal advantage to the link function, and then you can mitigate the performance hits by doing the one-time bindings as we did before with the custom compile function -- this will minimize the number of watches.


### Passing Parameters Out of a Directive

There may be times when you want to pass parameters out of a directive into a function, but how do you accomplish this?

example html:
<!DOCTYPE html>
<html>
  <head>
    <title>Controller Passing a Parameter</title>
  </head>
  <body>
    <div data-ng-app="directivesModule"
         data-ng-controller="CustomersController">
      <br>
      Parent Scope Customers:
      <br><br>
      <ul>
        <li ng-repeat="cust in customers">{{ cust.name }}</li>
      </ul>
      <br>
      <h3>Passing Parameters out of a Directive Controller</h3>
      <!-- In example 1 we have to pass in the name parameter to the function to get it to work -->
      Example 1: <div controller-passing-parameter1 datasource="customers" add="addCustomer(name)"></div>
      <br><br>
      <!-- In example 2 we're passing in a reference to the function, but we have to unwrap it, which is what we'll use vm.add() to do in the directive -->
      Example 2: <div controller-passing-parameter2 datasource="customers" add="addCustomer"></div>
      <br>
    </div>
    <script src="../../scripts/angular.js"></script>
    <script src="../../scripts/directivesController.js"></script>
    <script src="controllerPassingParameter1.js"></script>
    <script src="controllerPassingParameter2.js"></script>
  </body>
</html>

Recall that in the CustomersController, there is a function called addCustomer, which takes a parameter 'name'.
When we invoke addCustomer, how do we get data out of the directive and pass it as the parameter?

controllerPassingParameter1.js:
(function() {

  var controllerPassingParameter1 = function () {

    var controller = function () {

      var vm = this;

      init();

      function init() {
        vm.items = angular.copy(vm.datasource);
      }

      vm.addItem = function () {
        var name = 'New Customer Added by Directive: vm.add({name: name});';

        // our directive in html: 
        // '<div controller-passing-parameter2 datasource="customers" 
        // add="addCustomer(name)"></div>'

        // Call external scope's function -- addCustomer(name) -- but this
        // method is a little inflexible, because the parameter we pass in has
        // to match exactly the parameter in the 'add' attribute's function
        
        vm.add({name: name});

        //Add new item to directive scope
        vm.items.push({
          name: name
        });
      };
    },

    template = '<button ng-click="vm.addItem()">Add Item</button><ul>' +
               '<li ng-repeat="item in vm.items">{{ ::item.name }}</li></ul>';

    return {
      restrict: 'EA',
      scope: {
        datasource: '=',
        add: '&',
      },
      controller: controller,
      controllerAs: 'vm',
      bindToController: true,
      template: template
    };
  };

  angular.module('directivesModule')
    .directive('controllerPassingParameter1', controllerPassingParameter1);
}());

controllerPassingParameter2.js:
(function() {

  var controllerPassingParameter2 = function () {

    var controller = function () {

      var vm = this;

      init();

      function init() {
        vm.items = angular.copy(vm.datasource);
      }

      vm.addItem = function () {
        var name = 'New Customer Added by Directive: vm.add()(name)';

        // our directive in html: 
        // '<div controller-passing-parameter2 datasource="customers" 
        // add="addCustomer"></div>'

        // ^^^ Here we'll get a reference to the external function, then
        // invoke it, then pass in the name as its parameter:

        vm.add()(name);

        // ^^^ This is a little more flexible and maintainable, because we can
        // pass whatever we want as a parameter into the function.
        // It could be var foo, var apple, var whatever. 

        //Add new item to directive scope
        vm.items.push({
          name: name
        });
      };
    },

    template = '<button ng-click="vm.addItem()">Add Item</button><ul>' +
               '<li ng-repeat="item in vm.items">{{ ::item.name }}</li></ul>';

    return {
      restrict: 'EA',
      scope: {
        datasource: '=',
        add: '&',
      },
      controller: controller,
      controllerAs: 'vm',
      bindToController: true,
      template: template
    };
  };

  angular.module('directivesModule')
    .directive('controllerPassingParameter2', controllerPassingParameter2);
}());

Although in the above example we used a controllerAs property, we could do the same thing in a plain controller property just by changing vm to $scope, or in a link function as well.


### Understanding Transclusion







































































