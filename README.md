Angular Architecture Guide
==========================

_Opinionated Angular architecture guide for teams by @drpicox_

If you are looking for an opinionated guide for conventions, structures, and patterns programming large and scalable Angular applications, then step right in. This guide is based in many of my developments and close teams experience wiht Angular.

The purpose of this style guide is to provide guidance on building Angular applications by showing patterns and conventions and, more importantly, why I choose them.


Table of Contents
-----------------

1. [MVVM](#mvvm)
1. [Cohesion and coupling](#cohesion-and-coupling)
1. [Modules](#modules)
1. [Components naming](#components-naming)
1. [Components](#components)
1. [References between components](#references-between-components) // TODO
1. [Multi-transclusion](#multi-transclusion) // TODO
1. [Decorators](#decorators) // TODO
1. [Supports](#supports) // TODO
1. [Models](#models) // TODO
1. [Factories](#factories) // TODO
1. [States](#states) // TODO
1. [Services](#services) // TODO
1. [Providers](#providers) // TODO
1. [Remotes](#remotes) // TODO
1. [Storages](#storages) // TODO
1. [Configurations](#configurations) // TODO
1. [Helpers](#helpers) // TODO
1. [Routes](#routes) // TODO
1. [Handlers](#handlers) // TODO
1. [Styles](#styles) // TODO

MVVM
----

### MVC is not a Fashion
###### [Arch [X001](#arch-x001)]

  - Work with each MVC level separately.
  
    *Why?*: People tend to work with MVC columns: given each view create a 1 to 1 controller and model. It replicates code and lowes maintainability.

    *Why?*: Reuse models between models and controllers, reuse service controllers between view controllers. 

    *Why*?: Avoid to replicate code and add unnecesary coupling.

  ```javascript
  /* avoid */
  // src/app.route.details/detail.route.js
  angular
      .module('app.route.details')
      .config(detailsRoute);

  /* @ngInject */
  function detailRoute($routeProvider) {
    $routeProvider.when('/details/:id', {
        controller: DetailController,
        controllerAs: 'vm',
        templateUrl: 'src/app.route.details/detail.template.html',
        resolve: { detail: DetailResolver, },
    });
  }

  /* @ngInject */
  function DetailResolver(detailsService,$route) {
    return detailsService.get($route.current.params.id);
  }

  /* @ngInject */
  function DetailController(detail, detailsService) {
    this.detail = detail;
    this.service = detailsService;
    ...
  }

  // src/app.route.details/details.service.js
  angular
      .module('app.route.details')
      .factory('detailsService', detailsService);

  /* @ngInject */
  function detailsService($http) {
    var service = {
        addToCart = addToCart;
        get = get;
        ...
    };
    ...
  }

  // src/app.route.cart/cart.route.js
  angular
      .module('app.route.cart')
      .config(cartRoute);

  /* @ngInject */
  function cartRoute($routeProvider) {
    $routeProvider.when('/cart', {
        controller: CartController,
        controllerAs: 'vm',
        templateUrl: 'src/app.route.cart/cart.template.html',
    });
  }

  /* @ngInject */
  function CartController(cartService) {
    this.cart = cartService;
    ...
  }

  // src/app.route.cart/cart.service.js
  angular
      .module('app.route.cart')
      .factory('cartService', cartService);

  /* @ngInject */
  function cartService($http) {
    var service = {
        addProduct = addProduct;
        ...
    };
    ...
  }
  ```

  ```javascript
  /* recommended */
  // src/app.route.details/detail.route.js
  angular
      .module('app.route.details')
      .config(detailRoute);

  /* @ngInject */
  function detailRoute($routeProvider) {
    $routeProvider.when('/details/:productId', {
        controller: DetailController,
        controllerAs: 'vm',
        templateUrl: 'src/app.route.details/detail.template.html',
        resolve: { product: ProductResolver, },
    });
  }

  /* @ngInject */
  function ProductResolver(productsService,$route) {
    return productsService.get($route.current.params.productId);
  }

  /* @ngInject */
  function DetailController(product, cartService) {
    this.product = product;
    this.cart = cartService;
    ...
  }

  // src/app.route.cart/cart.route.js
  angular
      .module('app.route.cart')
      .config(cartRoute);

  /* @ngInject */
  function cartRoute($routeProvider) {
    $routeProvider.when('/cart', {
        controller: CartController,
        controllerAs: 'vm',
        templateUrl: 'src/app.route.cart/cart.template.html',
    });
  }

  /* @ngInject */
  function CartController(cartService) {
    this.cart = cartService;
    ...
  }

  // src/app.products/products.service.js
  angular
      .module('app.products')
      .factory('productsService', productsService);

  /* @ngInject */
  function productsService($http) {
    var service = {
        get = get;
        ...
    };
    ...
  }

  // src/app.cart/cart.service.js
  angular
      .module('app.cart')
      .factory('cartService', cartService);

  /* @ngInject */
  function cartService($http) {
    var service = {
        addProduct = addProduct;
        ...
    };
    ...
  }
  ```

  Note: Create an independent module for each kind of responsability, do not mix views with service controllers.


### Separe controllers and concerns
###### [Arch [X002](#arch-x002)]

  - Split controllers in viewmodel controllers and service controllers.
  
    *Why?*: View controllers should be as small as possible.

    *Why?*: Having service controllers apart allows to reuse them in multiple views.

    *Why?*: Lowes coupling and raises cohesion.

  ```javascript
  /* avoid */
  // src/app.route.cart/cart.route.js
  angular
      .module('app.route.cart')
      .config(cartRoute);

  /* @ngInject */
  function cartRoute($routeProvider) {
    $routeProvider.when('/cart', {
        controller: CartController,
        controllerAs: 'vm',
        templateUrl: 'src/app.route.cart/cart.template.html',
    });
  }

  /* @ngInject */
  function CartController($http) {
    var vm = this;
    vm.addProduct = addProduct;
    ...
  }
  ```

  ```javascript
  /* recommended */
  // src/app.route.cart/cart.route.js
  angular
      .module('app.route.cart')
      .config(cartRoute);

  /* @ngInject */
  function cartRoute($routeProvider) {
    $routeProvider.when('/cart', {
        controller: CartController,
        controllerAs: 'vm',
        templateUrl: 'src/app.route.cart/cart.template.html',
    });
  }

  /* @ngInject */
  function CartController(cartService) {
    var vm = this;
    vm.cart = cart;
    ...
  }

  // src/app.cart/cart.service.js
  angular
      .module('app.cart')
      .factory('cartService', cartService);

  /* @ngInject */
  function cartService($http) {
    var service = {
        addProduct = addProduct;
        ...
    };
    ...
  }
  ```


**[Back to top](#table-of-contents)**


Cohesion and coupling
---------------------

### Cohesion: keep related together 
###### [Arch [X010](#arch-x010)]

  - Keep in the same component close related things.

    *Why?*: Related things are usually subject to the same maintenance tasks.

    *Why?*: Avoid finding multiple files across your project when updating the same concept.

  ```javascript
  /* avoid */
  // src/app.profileviews/ProfileUpdate.controller.js
  angular
      .module('app.profileviews')
      .controller('ProfileUpdateController', ProfileUpdateController);

  /* ngInject */
  function ProfileUpdateController(profileService, $http) {
    this.profile = profileService;
    this.update = update;
  }

  // src/app.profile/ProfileUpdate.controller.js
  angular
      .module('app.profile')
      .factory('profileService', profileService);

  /* ngInject */
  function profileService($http) {
    var service = {
        username: '',
        ...
    };
    ...
  }
  ```

  ```javascript
  /* recommended */
  // src/app.profileviews/ProfileUpdate.controller.js
  angular
      .module('app.profileviews')
      .controller('ProfileUpdateController', ProfileUpdateController);

  /* ngInject */
  function ProfileUpdateController(profileService) {
    this.profile = profileService;
  }

  // src/app.profile/ProfileUpdate.controller.js
  angular
      .module('app.profile')
      .factory('profileService', profileService);

  /* ngInject */
  function profileService($http) {
    var service = {
        username: '',
        ...
        update: update,
    };
    ...
  }
  ```

  Note: Related things to keep together in the same file usually refers to the same concept.


### Cohesion: keep unrelated apart
###### [Arch [X011](#arch-x011)]

  - Keep apart different concepts or concerns.

    *Why?*: Each component implementing one single feature allows to avoid mixing concerns.

    *Why?*: Changes, updates, or documentation becomes better focused.

    *Why?*: It is easier and safer to change application polices when it is required.

  ```javascript
  /* avoid */
  // src/app.pictures/pictures.service.js
  angular
      .module('app.pictures')
      .factory('pircturesService', picturesService);

  /* ngInject */
  function picturesService($http,$storage) {
    var service = {
      get: get,
      ...
    }

    function get(id) {
      // get from storage, 
      // then if not saved get from remote
      // then save
      // then return result
      ...
    }
    ...
  }
  ```

  ```javascript
  /* recommended */
  // src/app.pictures/pictures.service.js
  angular
      .module('app.pictures')
      .factory('pircturesService', picturesService);

  /* ngInject */
  function picturesService(picturesStorage,picturesRemote) {
    var service = {
      get: get,
      ...
    }
    ...
  }

  // src/app.pictures/pictures.storage.js
  angular
      .module('app.pictures')
      .factory('pircturesStorage', picturesStorage);

  /* ngInject */
  function picturesStorage($storage) {
    var service = {
      get: get,
      save: save,
      ...
    }
    ...
  }

  // src/app.pictures/pictures.remote.js
  angular
      .module('app.pictures')
      .factory('pircturesRemote', picturesRemote);

  /* ngInject */
  function picturesRemote($http) {
    var service = {
      get: get,
      ...
    }
    ...
  }
  ```

  Note: It is easy to get lost in tones of lines of code trying to split future things, do not. Split components only when it is obvious that you are mixin two behaviours.

### Coupling: define a direction
###### [Arch [X012](#arch-x012)]

  - Define a direction of dependencies between components so you have a graph without cycles.

    *Why?*: Defining a direction of dependencies between components and modules creates SOLID architectures with better maintainability.

  ```
    ProfileViewController
      |
      |----> ProfileService
      |
      |                  -------> PictureRemote  --------> $http
      v                 |
    PictureService -----|
                        |
                         -------> PictureStorage --------> $storage
  ```

  Note: At least define directions between modules, and between components inside each module.

  ```
    app.profileview --> app.profile, app.pictures

    app.pictures:
                          -------> picturesRemote  --------> $http
                         |
    picturesService -----|
                         |
                          -------> picturesStorage --------> $storage
  ```

### Coupling: do not make circles
###### [Arch [X013](#arch-x013)]

  - Do not make cycles when programming, explicitly o implicitly. 

    *Why?*: Two components with a cycle of dependences become on single component when we are doing maintenance tasks.

    *Why?*: Usually when there is a cycle in fact there is a hidden component which undoes the cycle.

  ```
  /* avoid */

      Product <------- Cart
          |              ^
           --------------

  /* recommended */

      Product <-------- Cart
         ^                |
          --- LineItem <--
  ```

  Note: many times dependencies between components are not just injected with dependency injection but they are in fact assuming the existence of the other entity and manage it.

  ```javascript
  /* avoid */
  function Product(args) {
    this.id = args.id;
    this.cart = args.cart;
    this.count = 0;
  }
  ...

  function Cart() {
    this.products = [];
  }
  ```

  ```javascript
  /* recommended */
  
  function Product(args) {
    this.id = args.id;
  }

  function LineItem(args) {
    this.product = args.product;
    this.count = args.count || 0;
  }

  function Cart() {
    this.items = [];
  }
  ```

### Coupling: MVC with at least two kinds of controllers
###### [Arch [X014](#arch-x014)]

  - Split controllers in at least two kinds: Service controllers and View controllers.

  - Service controllers loads and saves data, and manages the state of the Application.
  
  - View controllers configures views using service controllers and should have minimum logic.
  
    *Why?*: View controllers should have minimum logic because the same models and informations can be presented in many ways (different kinds of views).

    *Why?*: Service controllers can be shared between multiple views.

  ```javascript
  /* avoid */
  // /src/app.booksview/BookEditController.js
  /* @ngInject */
  public BookEditController(book, $http) {
      var vm = this;
      vm.book = book;
      vm.save = save;
  }
  ```

  ```javascript
  /* recommended */
  // /src/app.booksview/BookEdit.controller.js
  angular
    .module('app.booksview')
    .controller('BookEditController', BookEditController);

  /* @ngInject */
  public BookEditController(book, booksService) {
      var vm = this;
      vm.book = book;
      vm.books = booksService;
  }

  // /src/app.books/books.service.js
  angular
    .module('app.books')
    .controller('booksService', booksService);

  /* @ngInject */
  public booksService($http) {
      var service = {
        save: save,
        ...
      };
      ...
  }
  ```

  Note: in the implementation booksService is published into the view as _books_ so a book can be saved just with `vm.books.save(vm.book)`.


### Coupling: MVC* dependence graph
###### [Arch [X015](#arch-x015)]

  - Only allow dependences between component kinds in the following directions: Models only knows other models; Views depends in models and other views; Service controller depends in other service controllers and models; and View controllers depends in all other services.

    *Why?*: Models should be the most simple component. So simple that they can be simple plain javascript objects, although they _implement_ an interface.

    *Why?*: Views should be easy to understand for designers, so they should kept as simple as possible.

    *Why?*: Service controllers should know nothing about views, because they are abstract entities that constructs the application state.

    *Why?*: View controllers have access to all elements because it is responsible to trigger communication between al levels. Although they have big couplings, view controllers should be really small so any change should have a minimum impact.

  ```
       -----------                ---------------------
      |   Views   |   <-------   |  View Controllers   |
       -----------                ---------------------
            |                /             |
            |              /               |
            |            /                 |
            |          /                   |
            v        L                     v
       -----------                ---------------------
      |  Models   |   <-------   | Service Controllers |
       -----------                ---------------------
  ```

  Note: in fact, as seen in previous examples, views uses service controllers, but it can be also interpreted as a shortcut of the view definition that have callbacks that triggers the view controller, which executes a service controller.


### Coupling: Composition bettern than inheritance
###### [Arch [X016](#arch-x016)]

  - When you can choose between composition or inheritance choose composition.

    *Why?*: Inheritance has a high coupling between parent and child class.

    *Why?*: A change in the parent usually affects to child class.

    *Why?*: Composition allows to implement easily multiple interfaces.

    *Why?*: Javascript do not have classes like other common Object Oriented programming languages, neither has inheritance like other programming languages.

    *Why?*: Class casting does not limits Javascript behaviour; you can assume that an Object satisfies the interface.

  ```javascript
  /* avoid */
  // src/app.engine/entity.model.js
  angular
    .module('app.engine')
    .value('Entity', Entity);

  function Entity(data) {
    angular.extends(this, data);
  }
  Entity.prototype.collide = collide;
  Entity.prototype.draw = draw;
  Entity.prototype.move = move;
  ...

  // src/app.engine/ball.model.js
  angular
    .module('app.engine')
    .factory('Ball', ballFactory);

  /* @ngInject */
  function ballFactory(Entity) {
    function Ball(data) {
      Entity.call(this, data);
    }
    Ball.prototype = new Entity();
  }
  ```

  ```javascript
  /* recommended */
  // src/app.engine/entity.model.js
  angular
    .module('app.engine')
    .value('Entity', Entity);

  function Entity(data) {
    angular.extends(this, data);
  }
  ...

  // src/app.engine/solid.model.js
  angular
    .module('app.engine')
    .value('Solid', Solid);

  function Solid(data) {
    angular.extends(this, data);
  }
  ...

  Solid.prototype.collide = collide;

  // src/app.engine/visible.model.js
  angular
    .module('app.engine')
    .value('Visible', Visible);

  function Visible(data) {
    angular.extends(this, data);
  }
  ...

  // src/app.engine/movable.model.js
  angular
    .module('app.engine')
    .value('Movable', Movable);

  function Movable(data) {
    angular.extends(this, data);
  }
  Movable.prototype.move = move;
  ...

  // src/app.engine/ball.model.js
  angular
    .module('app.engine')
    .factory('Ball', ballFactory);

  /* @ngInject */
  function ballFactory(Entity, Solid, Visible, Movable) {
    function Ball(data) {
      Entity.call(this, data);
      Solid.call(this, data);
      Visible.call(this, data);
      Movable.call(this, data);
    }
    angular.extend(Ball.prototype, Entity.prototype);
    angular.extend(Ball.prototype, Solid.prototype);
    angular.extend(Ball.prototype, Visible.prototype);
    angular.extend(Ball.prototype, Movable.prototype);
  }
  ```

  Note: if you can you may use properties.

  ```javascript
  /* even better */
  // src/app.engine/ball.model.js
  angular
    .module('app.engine')
    .factory('Ball', ballFactory);

  /* @ngInject */
  function ballFactory(Entity, Solid, Visible, Movable) {
    function Ball(data) {
      Entity.call(this, data);
      this.solid = new Solid();
      this.visible = new Visible();
      this.movable = new Movable();
    }
    angular.extend(Ball.prototype, Entity.prototype);
  }
  ```


### Coupling: feature detection better than polymorphism
###### [Arch [X017](#arch-x017)]

  - Avoid classic polymorphism in benefit of feature detection. Instead of creating classes and polymorphic methods or polymorphic views relay on detecting features (properties and methods) present in each object to choose if given controller or view acts or not.

    *Why?*: Handle polymorphism creates a high coupling of the factory of the polymorphic object with all possible subclasses.

    *Why?*: Polymorphism usually forces to know 100% the characteristics of an Object since the first moment instead of allow to construct it in multiple steps.

    *Why?*: Root _class_ tends to integrate all kind of services and stubs related to its children.

    *Why?*: Polymorphism hides and mades unclear the final behaviour of the component.

    *Why?*: Javascript functional behaviour is the final polymorphic class with a single polymorphic method.

  ```html
  <!-- avoid -->
  <!-- src/app.productsview/productDetail.component.html -->
  <div class="product" ng-include="'src/app.productsview/productDetail-'+vm.product.kind+'.component.html'">
  </div>
  ```

  ```html
  <!-- recommended -->
  <!-- src/app.productsview/productDetail.component.html -->
  <div class="product">
    <h1 ng-bind="vm.product.name"></h1>
    <product-price data-product="vm.product" ng-if="!!vm.product.price"></product-price>
    <product-supplies data-product="vm.product" ng-if="!!vm.product.supplies"></product-supplies>    
  </div>
  ```


**[Back to top](#table-of-contents)**
 
Modules
-------

### Modules directories
###### [Arch [X020](#arch-x020)]

  - Put each module and its components inside a directory with the module name inside the `src/` directory. Yes, directories can have dots.

    *Why?*: The `src/` directory separes source to be developed and versioned from other libraries or configurations.

    *Why?*: Instead of creating nested directories making all directories at the same levels is useful to get a good overview of the application.

    *Why?*: Easy to locate each directory.

  ```
  - avoid.txt
  - src/
    + common/
    - products/
      + routes/
      + services/
      + views/
    - users/
      + routes/
      + services/
      + views/
  ```

  ```
  - recommended.txt
  - src/
    + app/
    + app.products.services/
    + app.products.views/
    + app.routes.products/
    + app.routes.users/
    + app.users.services/
    + app.users.views/
    + common/
  ```


### App module built with children module
###### [Arch [X021](#arch-x021)]

  - Keep app module almost empty with just the required configurations specific for the app and dependencies with all children modules. Routes, views, services, and other componentes should be each in its module.

    *Why?*: We are targeting large developments.

    *Why?*: We want to keep close related things, and far unrelated.

    *Why?*: Files in a directory should not exceed 10 in average, 20 most.

  ```
  - avoid.txt
  - src/app.module.js
  - src/users.config.js
  - src/users.route.js
  - src/users.service.js
  ```

  ```
  - recommended.txt
  - src/app/app.module.js
  - src/app/users.config.js
  - src/app.routes.users/routesUsers.module.js
  - src/app.routes.users/users.route.js
  - src/app.users.services/servicesUsers.module.js
  - src/app.users.services/users.service.js
  ```


### Separe concerns in modules
###### [Arch [X022](#arch-x022)]

  - Separe routes, view components, and services and models in different modules.

    *Why?*: View components are usually shared between multiple routes.

    *Why?*: Services and models are related, and they are usually shared across any other module.

    *Why?*: Reduce accidental coupling: spliting them in many modules it is easier to follow Cohesion and Coupling principles.

  ```
  - avoid.txt
  - src/app/app.module.js
  - src/app/users.config.js
  - src/app.users/userDetail.component.js
  - src/app.users/users.module.js
  - src/app.users/users.route.js
  - src/app.users/users.service.js
  - src/app.users/User.model.js
  ```

  ```
  - recommended.txt
  - src/app/app.module.js
  - src/app/users.config.js
  - src/app.rotutes.users/routesUsers.module.js
  - src/app.rotutes.users/users.route.js
  - src/app.users.views/userDetail.component.js
  - src/app.users.views/usersViews.module.js
  - src/app.users.services/User.model.js
  - src/app.users.services/users.service.js
  - src/app.users.services/usersServices.module.js
  ```


**[Back to top](#table-of-contents)**


Components naming
-----------------

### Put role in the name
###### [Arch [X030](#arch-x030)]

  - For each component suffix the name with the role. Do not limit to _directive_, _service_, _model_, _route_, _run_, _controller_, but instead use more detailed names. Define carefully names and roles and use them consistenly across all your project. For example _component_ or _widget_ for directives that are reusable visual components, _decorator_ for directives that changes the behaviour of a template widget, _state_ for those services that keeps a state but do not access remote, _storage_ for services that saves data in the client, _remote_ for services that handles a remote connection, ...

    *Why?*: It allows to create a better separation of concerns.

    *Why?*: It allows to recognize more easily the task of each component.

  ```
  - avoid.txt
  - src/app.users.services/usersStorage.service.js
  - src/app.users.services/users.service.js
  - src/app.users.services/User.model.js
  ```

  ```
  - recommended.txt
  - src/app.users.services/users.remote.js
  - src/app.users.services/users.service.js
  - src/app.users.services/users.storage.js
  - src/app.users.services/user.model.js
  ```

  Note: in this case the users service creates the bridge between remote and storage.

  Note: ex: users.remote.js define usersRemote service.

  Note: in this document there is already a recommendation.


**[Back to top](#table-of-contents)**

Components
----------

Components are those directives that have a controller and a template, with or without transclude.

### Read carefully John Papa style

  - Provide a Unique Directive Prefix
  See: https://github.com/johnpapa/angular-styleguide#style-y073

  - Directives and ControllerAs
  See: https://github.com/johnpapa/angular-styleguide#style-y075

  - Bind scope to controller
  See: https://github.com/johnpapa/angular-styleguide#style-y076


### Restrict components to entity only 
###### [Arch [X040](#arch-x040)]

  - Make components directives restricted to entity only.

    *Why?*: It is easy to recognize components in template code.

  ```javascript
  /* avoid */
  // src/app.users.views/userDetail.component.js
  angular
    .module('app.users.view')
    .directive('appUserDetail', userDetail);

  function userDetail() {
    var directive = {
      restrict: 'A',
      controller: UserDetailController,
      controllerAs: 'vm',
      scope: {},
    };
    return directive;
  }
  ...
  ```

  ```javascript
  /* recommended */
  // src/app.users.views/userDetail.component.js
  angular
    .module('app.users.view')
    .directive('appUserDetail', userDetail);

  function userDetail() {
    var directive = {
      restrict: 'E',
      controller: UserDetailController,
      controllerAs: 'vm',
      scope: {},
    };
    return directive;
  }
  ...
  ```

  Note: if your are using Angular 1.2 and your target browser includes Internet Explorer 8, you may use `restrict: 'A'`, but suffix the directive with View.

  ```javascript
  /* for IE8 */
  // src/app.users.views/userDetail.component.js
  angular
    .module('app.users.view')
    .directive('appUserDetailView', userDetail);

  function userDetail() {
    var directive = {
      restrict: 'A',
      controller: UserDetailController,
      controllerAs: 'vm',
      scope: {},
    };
    return directive;
  }
  ...
  ```


### Use data- in the scope attributes 
###### [Arch [X041](#arch-x041)]

  - In the template, when you are passing attributes to a directive, use `data-` preffix.
 
    *Why?*: It is easy to recognize that these attributes are directives arguments and not other directives themselves.

  ```html
  <!-- avoid -->
  <product-detail product="product">
  </product-detail>
  ```

  ```html
  <!-- recommended -->
  <product-detail data-product="product">
  </product-detail>
  ```

  Note: this convention should be applied also to decorators.

### Use isolated scopes
###### [Arch [X042](#arch-x042)]

  - Use always isolated scopes in components.

    *Why?*: It avoids side effects in the interaction between scopes.

  ```javascript
  /* avoid */
  // src/app.users.views/userDetail.component.js
  angular
    .module('app.users.view')
    .directive('appUserDetail', userDetail);

  function userDetail() {
    var directive = {
      restrict: 'E',
      controller: UserDetailController,
      controllerAs: 'vm',
      scope: true,
    };
    return directive;
  }
  ...
  ```

  ```javascript
  /* recommended */
  // src/app.users.views/userDetail.component.js
  angular
    .module('app.users.view')
    .directive('appUserDetail', userDetail);

  function userDetail() {
    var directive = {
      restrict: 'E',
      controller: UserDetailController,
      controllerAs: 'vm',
      scope: {},
    };
    return directive;
  }
  ...
  ```


### Directive and controllers together
###### [Arch [X043](#arch-x043)]

  - Keep directives and controllers in the same file.

    *Why?*: Directive is just a definition, controller has the behaviour. Splitting both creates almost meaningless files (directive configuration) separated from all the relevant logic.

    *Why?*: It is very close to the Angular 2.0 logic, so transition should be easier.

  ```javascript
  /* avoid */
  // src/app.users.views/userDetail.component.js
  angular
    .module('app.users.view')
    .directive('appUserDetail', userDetail);

  function userDetail() {
    var directive = {
      restrict: 'E',
      controller: 'UserDetailController',
      controllerAs: 'vm',
      scope: {},
    };
    return directive;
  }
  
  // src/app.users.views/userDetail.controller.js
  angular
    .module('app.users.view')
    .controller('UserDetailController', UserDetailController);

  /* @ngInject */
  function UserDetailController(userService) {
    var vm = this;
    vm.user = userService;
  }
  ```

  ```javascript
  /* recommended */
  // src/app.users.views/userDetail.component.js
  angular
    .module('app.users.view')
    .directive('appUserDetail', userDetail);

  function userDetail() {
    var directive = {
      restrict: 'E',
      controller: UserDetailController,
      controllerAs: 'vm',
      scope: {},
    };
    return directive;
  }

  /* @ngInject */
  function UserDetailController(userService) {
    var vm = this;
    vm.user = userService;
  }
  ```

  Note: using this schema controllers are completely private for each file and not published.

  Note: although because testability it seems better to split, so controller can be tested without the directive, the controller is responsible for the directive behavior, so the controller should be tested through the directive, which is the controller exposed.

  Note: using IFEE, browserify, ... UserDetailController becomes a private function of the current scope, so it can be mangled and minimized.

### Styles and templates together
###### [Arch [X044](#arch-x044)]

  - Keep styles and templates in the same directory.

    *Why?*: Styles and templates have a strong dependence, it is very likely that the developer/designer will change both together. 

    *Why?*: Template is the view, it is configured by the controller. So there is a strong dependence between both.

  ```
  - avoid.txt
  - src/app.cart.views/cart.component.js
  - src/styles/cart.style.less
  - src/templates/cart.tpl.html
  ```

  ```
  - recommended.txt
  - src/app.cart.views/cart.component.js
  - src/app.cart.views/cart.style.less
  - src/app.cart.views/cart.tpl.html
  ```


### Style elements
###### [Arch [X045](#arch-x045)]

  - When defining a style for a component do it over the css element, no over a css class.
 
    *Why?*: Applying styles over classes forces to put the class each time that the directive is used.

  ```css
  /* avoid */
  .product-detail {
    display: block;
    ...
  }
  ```

  ```css
  /* recommended */
  [product-detail] {
    display: block;
    ...
  }
  ```


### Link and activate
###### [Arch [X046](#arch-x046)]

  - Do not use link.

  *Why?*: The objective of link is to manipulate DOM, or to execute code after children components are initialized. DOM manipulation should be done with mainly with decorators. To execute acode after children are initialized we use `activate` method in the controller (called by link).

  *Why?*: Using `link` usually tends to use scope erroniously.

  ```javascript
  /* avoid */
  // src/app.users.views/userDetail.component.js
  angular
    .module('app.users.view')
    .directive('appUserDetail', userDetail);

  function userDetail() {
    var directive = {
      restrict: 'E',
      controller: UserDetailController,
      controllerAs: 'vm',
      scope: {},
      link: link;
    };
    ...
    return directive;
  }
  ...
  ```

  ```javascript
  /* recommended */
  // src/app.users.views/userDetail.component.js
  angular
    .module('app.users.view')
    .directive('appUserDetail', userDetail);

  function userDetail() {
    var directive = {
      restrict: 'E',
      controller: UserDetailController,
      controllerAs: 'vm',
      scope: {},
    };
    return directive;
  }
  ...
  ```


###### [Arch [X047](#arch-x047)]

  - Use link for activate. It will call activate from controller.

    *Why?*: Some initializations should be done after all children are initialized. Controller is initialized before children, but link after children.

    *Why?*: Activation code should remain in the controller.

  ```javascript
  /* avoid */
  // src/co.widgets.slider/coSlider.component.js
  angular
    .module('co.widgets.slider')
    .directive('coSlider', slider);

  function slider() {
    var directive = {
      restrict: 'E',
      controller: SliderController,
      controllerAs: 'vm',
      scope: {},
      link: link;
    };
    
    return directive;

    function link (scope) {
      angular.forEach(scope.vm.items, function(item) {
        ...
      });
    }
  }
  
  function SliderController() {
    var vm = this;
    vm.items = [];
  }
  ```

  ```javascript
  /* recommended */
  // src/co.widgets.slider/coSlider.component.js
  angular
    .module('co.widgets.slider')
    .directive('coSlider', slider);

  function slider() {
    var directive = {
      restrict: 'E',
      controller: SliderController,
      controllerAs: 'vm',
      scope: {},
      link: link;
    };
    
    return directive;

    function link (scope) {
      scope.vm.activate();
    }
  }
  
  function SliderController() {
    var vm = this;
    vm.items = [];
    vm.activate = activate;

    function activate() {
      angular.forEach(vm.items, function(item) {
        ...
      });
    }
  }
  ```


**[Back to top](#table-of-contents)**


References between components
-----------------------------

### Use #id decorator
###### [Arch [X050](#arch-x050)]

  - Use an `id` decorator to emulate Angular2 `#id` notation.

    *Why?*: It allows to create more flexible relationships between directives.

    *Why?*: Using `ng-controller` induces poor decisions and it has hard limits: maximum 1 ng-controller per entity, only children entities can see controllers, no arguments are allowed to parametrize them.

  ```html
  <!-- avoid -->
  <div ng-controller="LoadDataController as loader">
    <div ng-if="loader.loading">Loading...</div>
    <div ng-if="!loader.loading" ng-bind="loader.data"></div>
  </div>
  ```


  ```html
  <!-- recommended -->
  <data-loader id="loader"></data-loader>
  <div ng-if="$.loader.loading">Loading...</div>
  <div ng-if="!$.loader.loading" ng-bind="$.loader.data"></div>
  ```


  Note: you can find here https://github.com/drpicox/drpx-id an implementation of the `id` decorator.


**[Back to top](#table-of-contents)**


- do not use link but activate (link is for decorators)
- id 
- transcludes
- controller


Decorator: directiva d’atribut que canvia comportament sense scope
Model:
Factory: creador de models amb nom (casi com state)
State / Collection: guarda collections i te $update
Service: fa IO
Support: a directive amb dom...
Helper: functions reusables
Tool: no angular