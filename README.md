Angular Architecture Guide
==========================

_Opinionated Angular architecture guide for teams by @drpicox_

If you are looking for an opinionated guide for conventions, structures, and patterns programming large and scalable Angular applications, then step right in. This guide is based in many of my developments and close teams experience wiht Angular.

The purpose of this style guide is to provide guidance on building Angular applications by showing patterns and conventions and, more importantly, why I choose them.


Table of Contents
-----------------

1. [MVVM](#mvvm)
1. [Cohesion and coupling](#cohesion-and-coupling)
1. [Modules](#modules) // TODO
1. [Components naming](#components-naming) // TODO
1. [Components](#components) // TODO
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

**[Back to top](#table-of-contents)**
  

Views Models ViewControllers ServiceControllers
Coupling: define a strategy of usage
Coupling: Composition bettern than inheritance

Component: directiva de entitat
Decorator: directiva d’atribut que canvia comportament sense scope
Model:
Factory: creador de models amb nom (casi com state)
State / Collection: guarda collections i te $update
Service: fa IO
Support: a directive amb dom...
Helper: functions reusables
Tool: no angular