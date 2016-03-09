Here I'm going to explain a bit my work methodologies and my coding style

# Javascript/Angular

When I code Angular, I always try to follow a simple folders structure
* Assets
  * This is for the external elements and the client-side code. E.g., bower_components, css, jQuery plugins...
* Components
  * Here I save the specific modules for a particular task, and the less reusable ones. E.g. If we are doing an app to show our remaining tasks, here goes the module that loads the list and work with it.
* Shared
  * This folder is for the reusable elements in other views. E.g. The sidebar directive, the topbar directive, modal views service...
* App: the main module of the app, where the dependencies are injected.
* Index: the main view of the app.

Why I do this? Because I get more scalability and the only thing I need to do when the project needs to be extended is to add a module to the correct folder with his elements (styles and other components) and inject it.  Also, is a logic and comprehensible way to have the elements stored to improve the maintenance. Every single folder have different contents but the main structure is to have the controller and the view or model.

Some code snippets:
```html
<!DOCTYPE html>
<html lang="en" ng-app="awesomeWebApp">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    ...
    <-- Meta tags -->

    <title>World Domination Management WebApp</title>

    <!-- Bootstrap core CSS -->
    <link href="assets/css/bootstrap.css" rel="stylesheet">
    <!--external css-->
    <link href="assets/font-awesome/css/font-awesome.css" rel="stylesheet" />

    <!-- Custom styles -->
    <link href="assets/css/style.css" rel="stylesheet">
    <link href="assets/css/style-responsive.css" rel="stylesheet">

    <!-- Older browsers compatibility -->
    <!-- HTML5 shim and Respond.js IE8 support of HTML5 elements and media queries -->
    <!--[if lt IE 9]>
      <script src="https://oss.maxcdn.com/libs/html5shiv/3.7.0/html5shiv.js"></script>
      <script src="https://oss.maxcdn.com/libs/respond.js/1.4.2/respond.min.js"></script>
    <![endif]-->

    <!-- Angular and jQuery (if necessary, but I prefer to use just Angular functions and jQuery Lite) -->
    <script src="assets/js/jquery.js"></script>
    <script src="assets/bower_components/angular/angular.min.js"></script>
  </head>

  <!-- In this case, I use an attribute directive called theme to load dynamically themes, and then I add some code to see how the directives are structured -->
  <body theme>
    <!-- Web preload -->
    <div id="loader-wrapper">
        <div id="loader"></div>
        <div class="loader-section section-left"></div>
        <div class="loader-section section-right"></div>
    </div>
    <!-- End of the preload -->

  <!-- The core directives of the app -->
  <section id="container">
      <topbar></topbar>
      <sidebar></sidebar>
      <section id="main-content">
          <section class="wrapper">
            <stats></stats>
            <todo-list></todo-list>
        </section>
      </section>
      <bottombar></bottombar>
  </section>

    <!-- Javascript code. TODO: learn RequireJS to optimize the loading performance of this components -->
    <script src="assets/js/bootstrap.min.js"></script>
    <script class="include" type="text/javascript" src="assets/js/jquery.xxx.1.5.js"></script>
    <script src="assets/js/jquery.scrollTo.min.js"></script>

    <!-- Main controllers -->
    <script src="app.js"></script>
    <script src="assets/bower_components/ngBlabla/src/blabla.js"></script>
    <script src="assets/bower_components/ngUtil/util.js"></script>
    <script src="components/list/todoController.js"></script>
    <script src="shared/storageService/storageService.js"></script>
    <script src="shared/themes/themesLoaderController.js"></script>

    <!-- Charts and controller -->
    <script src="assets/js/stats/stats.min.js"></script>
    <script src="components/stats/statsController.js"></script>
    <script src="shared/sidebar/sidebarController.js"></script>
    <script src="shared/topbar/topbarController.js"></script>
    <script src="shared/bottombar/bottombarController.js"></script>

    <!-- Some common scripts of the page, E.g. add parameters to chartlist, initialize nicescroll.js... -->
    <script src="assets/js/common-scripts.js"></script>

    <!-- In this case, here i control the preload, this is an specific case -->
    <script>
    $(document).ready(function() {
    	setTimeout(function(){
    		$('body').addClass('loaded');
    	}, 3000);
    });
    </script>

  </body>
</html>
```
NOTE: this is a develop code. In production I will compress all the controllers, modules, styles, etc. and uglify them, the views to the template cache...

Now a pure Angular code:

```javascript
/*
  With this controller, the colors of the page theme are updated as
  specified by the user
*/
(function(){
    var app = angular.module("Themes", ["StorageService"]);

    app.controller("ThemesController", ["StorageManager", "$scope", function(StorageManager, $scope){
      /* To optimize the content load between pages, I decided to use a module called ngStorage and I created a service to improve it maintenance. This is an old code, but now I always try to avoid to use $scope and always tend to use "controller as" and a view model (vm) to set the attributes and other stuff, always using the angular way notation. E.g:

     var vm = this;
     vm.foo = 'bar';
     vm.someCoolFunction = myCoolFunction;

     function myCoolFunction() {
       ...
       return 0;
     }
 */

      this.titleColor = StorageManager.loadTitleColor();
      this.backgroundColor = StorageManager.loadBackgroundColor();
      this.sidebarColor = StorageManager.loadSidebarColor();

      /*
        Define the js object with the properties of the theme
      */
      $scope.json = {
            "theme": [
              {
                  "element": "body",
                  "attribute": "background-color",
                  "value": this.backgroundColor
              },
              {
                  "element": ".mainTitle",
                  "attribute": "color",
                  "value": this.titleColor
              },
              {
                  "element": "#sidebar",
                  "attribute": "background-color",
                  "value": this.sidebarColor
              }
              ...
          ]
      };

      /* In this case, I did it in a more traditional way, because the algorithm that uses Angular to load directives didn't able me to load correctly the themes dynamically, and some views wasn't loaded when this method was executed */

      /*
      *  Assign the values of a theme with a given json.
      *  @param json The json with the elements and the colors to edit
      */
      $scope.assignValues = function(){
        var style = "<style>"
          for(var i=0; i < $scope.json.theme.length; i++){
            var element = $scope.json.theme[i].element;
            var attribute = $scope.json.theme[i].attribute;
            var value = $scope.json.theme[i].value;
            style += element + "{" + attribute + ":#" + value + ";}";
        }
        style += "</style>";
        $("head").append(style);
      }
    }]);


    /* This code was an initial aproximation, and did not work as well as I thought. Theoretically It should work but, at the end of the day, the elements did not load correctly the theme in the final elements

   Also, this is an old code and some stuff that I did here probably in this moment I would do it differently. For example, to load the values of the theme now I would modularize the view in a lot of directives with isolated scopes:*/

  <custom-element custom-style="data.customStyles.customElement"></custom-element>

/* and in the view, I would use a ng-style to add the styles in the element. Example: */

  <div class="wrapper-cool-text uppercase bold center active" ng-style="vm.style.mainText">...</div>

    /*
    * With this directive I assure that the theme is loaded
    * at the end of the theme, when the rest of the DOM elements
    * are finished it's generation
    */
    app.directive('theme', [function() {
      return {
        priority: Number.MIN_SAFE_INTEGER,
        restrict: 'A',
        controller: "ThemesController",
        link: function($scope) {
          $scope.assignValues();
        }
      };
    }]);

   /* Of course, now I declare the directives on other way. I always tend to follow the good practices of an Angular Guru like John Papa to improve my conding -> https://github.com/johnpapa/angular-styleguide . An example: */

(function() {
    'use strict';

    angular
        .module('module')
        .directive('directive', directive);

    /* @ngInject */
    function directive() {
        var directive = {
            restrict: 'EA',
            templateUrl: 'templateUrl',
            scope: {
            },
            link: linkFunc,
            controller: Controller,
            controllerAs: 'vm',
            bindToController: true
        };

        return directive;

        function linkFunc(scope, el, attr, ctrl) {

        }
    }

    Controller.$inject = ['dependencies'];

    /* @ngInject */
    function Controller(dependencies) {
        var vm = this;

        activate();

        function activate() {

        }
    }
})();

  /* Maybe seems too hard to remember or so strange, but I use Atom as my main IDE and I've this declaration and other elements, such Controller, as snippet. For me, the hardest part is typing 'ngDir', and it's done :3 */
})();

```

This is a code snippet of a test I did on TestDome (pure JS). The exercise was about introducing a date and with a format and modify it's order.

```javascript
function formatDate(userDate) {
   // format from M/D/YYYY to YYYYMMDD
  var date = [],
      strLength = userDate.length,
      convertedDate = "";

  date = userDate.split("/", 3);
  convertedDate += date[2];

  if(date[0].length == 1) {
	convertedDate += "0" + date[0];
  }
  else if(date[0].length == 2) {
    convertedDate += date[0];
  }

  if(date[1].length == 1) {
    convertedDate += "0" + date[1];
  }
  else if(date[1].length == 2) {
    convertedDate += date[1];
  }

  return convertedDate;
}
```

This is old, but I think is interesting as a way to see how I code pure JS. I tend to enclosure every if-else function, because is more clear than the if(true) myFunction(); else otherFunction(); when I have a lot of statements. Also, I'm currently trying to adapt my JS to the standard style (https://github.com/feross/standard). Finally, my way of work with every JS module could be the following:

```javascript
(function() {
  var foo = foo;

  function foo() {
    var vm = this;
    vm.bar = null;
    vm.yo = 'Hey';
  };

  foo.prototype.niceFunction = niceFunction;

  function niceFunction(elem, params) {
   ...
   return 0;
  }

  var bar = new foo();
}());
```

# PHP/Laravel

I'm going to add a snippet of pure PHP and a Laravel snippet. The architecture in the last case is 100% MVC.

## PHP

```php
class Palindrome
{
    public static function isPalindrome($str)
    {
return self::check($str);
    }

    public static function check($str){
        $a = strtolower($str);

        $conservar = '0-9a-z';
        $regex = sprintf('~[^%s]++~i', $conservar);
        $a = preg_replace($regex, '', $a);
        $b = strrev($a);

        if($a == $b)
       return true;            
        else
        return false;
   }
}
```

## Laravel
### This is for creating an excel with some users from the DB

```php
<?php

class ExcelController extends Controller {

  public function makeExcel() {

    //With an array given with the colums to use in the excel,
    //The query is made
    $columns = array(
      'worldDominationTable.id',
      'worldDominationTable.name',
      'worldDominationTable.surname',
      'worldDominationTable.email'
    );

    //If there is one or more colums, the query is created.Si se pasa 1 o más columnas se crea la consulta
    if(sizeof($columns)> 0){
      $select = $columns[0];
      for ($i = 1; $i <sizeof($columns); $i++) {
        $select .= ", $columnsToShow[$i]";
      }
    }

    $users = DB::table('worldDominationTable')
      ->select(DB::raw("$select"))
      ->get();

    //If the query gets one or more rows, the excel is created
    if(sizeof($users) > 0){
      //Dependancy injections
      $excel = App::make('excel');

      Excel::create('Users', function($excel) use($users, $columnsToShow) {
        $excel->sheet('Users', function($sheet) use($users, $columnsToShow) {

          $arrayManager = new ArrayManager();

          //Get the keys
          $headers = $arrayManager->getKeys($users);
          //Get an array with the users
          $excel  = $arrayManager->createArray($headers, $users);
          //Create the excel
          $sheet->fromArray($excel, null, 'A1', true, false);
          //Set the headers
          $sheet->prependRow(1, $headers);

          //Format: Helvetica 14px
          $sheet->setStyle(array(
            'font' => array(
            'name'      =>  'Helvetica Neue',
            'size'      =>  14
            )
          ));

          //The header in Bold
          $sheet->cells('A1:O1', function($cells) {
            $cells->setFontWeight('bold');
          });

        });
      })->download('xls');

      return Response::json(array('success' => true));
    }else{
      return Response::json(array('success' => false));
    }
  }
}
```
# Android
## Method to filter a list of users

```java
    public void filter(String charText) throws JSONException {
        charText = charText.toLowerCase(Locale.getDefault());
        String fullname;
        usersList.clear();
        if (charText.length() == 0) {
            usersList.addAll(arraylist);
        }
        else{
            for(int i=0; i<arraylist.size(); i++){
                fullname = arraylist.get(i).getString("name").toLowerCase(Locale.getDefault()) +" "+ arraylist.get(i).getString("surname").toLowerCase(Locale.getDefault());
                if (fullname.contains(charText)){
                    usersList.add(arraylist.get(i));
                }
                else if (arraylist.get(i).getString("email").toLowerCase(Locale.getDefault()).contains(charText)){
                    usersList.add(arraylist.get(i));
                }
                else if (arraylist.get(i).getString("code").toLowerCase(Locale.getDefault()).contains(charText)){
                    usersList.add(arraylist.get(i));
                }
            }
        }
        notifyDataSetChanged();
    }
```
# iOS
## Generate a login petition and save the received elements.

This is an old one, and now I would code Swift instead of Objective-C, because of it's simplicity, power, speed and security (I've done a course at TeamTreehouse about Swift but I've not used in any commercial project)

```objective-c
- (NSMutableURLRequest*)GenerateRequest: (NSString *) Url : (NSString *) Post {

    NSString *post = Post;
    NSURL *url = [NSURL URLWithString:Url];

    NSData *postData = [post dataUsingEncoding:NSASCIIStringEncoding allowLossyConversion:YES];

    NSString *postLength = [NSString stringWithFormat:@"%lu", (unsigned long)[postData length]];

    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] init];

    [request setURL:url];
    [request setHTTPMethod:@"POST"];
    [request setValue:postLength forHTTPHeaderField:@"Content-Length"];
    [request setValue:@"application/json" forHTTPHeaderField:@"Accept"];
    [request setValue:@"application/x-www-form-urlencoded" forHTTPHeaderField:@"Content-Type"];
    [request setHTTPBody:postData];

    return request;
}

- (void)saveData: (NSData *) DataReceived {
    NSError *error = nil;
    NSDictionary *UserPreferences = [NSJSONSerialization
                              JSONObjectWithData:DataReceived
                              options:NSJSONReadingMutableContainers
                              error:&error];

    NSUserDefaults *defaults = [NSUserDefaults standardUserDefaults];

    [defaults setObject:UserPreferences forKey:@"UserPreferences"];

    [defaults synchronize];
}
```
# Sass/Scss

I love to create a master partial where I'm loading everything I need, and this is the one which is imported to the main style, usually style.scss.

```scss
//Import the colors and the typefaces
@import "colors";
@import "typefaces";

//Bourbon framework
@import "bourbon/bourbon";
@import "base/base";
@import "neat/neat";

//Import the rest of the elements
@import "stats";
```

Also I like, for example, to define all the elements I'm going to use in a partial, and then assign them to other variable that will be used in the code. Thanks to this, I can manage so fast the changes of the theme, and prevent the inconsistencies of modifying a color or a font-size in the whole project, for example. Snippet:

```scss
$orange: #E8940C;
$red: #E82C0C;
$yellow: #CFFF0D;
$green: #00B233;
$black: #292a2c;

//Accents
$color_accent: $green;
$color_accent_secondary: $red;

//Titles
$color_title: $black;
$color_title_secondary: $green;

//Bars
$color_bar_main: $yellow;
$color_bar_secondary: $orange;
```

And define the convenient mixins:

```scss
//Methods to apply colors
@mixin font-border($font, $border){
    color: $font;
    border-color: $border;
}

@mixin font-background($font, $background){
    color: $font;
    background-color: $background;
}

@mixin font-background-border($font, $background, $border){
    color: $font;
    background-color: $background;
    border-color: $border;
}
```

I like to use Bourbon as Sass library, but currently in my job I'm using Compass in a Gulp dependency. Finally, I'm so interested in the BEM architecture for my projects when coding Sass (http://www.sitepoint.com/bem-smacss-advice-from-developers/)
