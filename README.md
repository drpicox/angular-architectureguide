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
1. [References between components](#references-between-components)
1. [Multi-transclusion](#multi-transclusion)
1. [Decorators](#decorators)
1. [Supports](#supports)
1. [Models](#models)
1. [Factories](#factories)
1. [States](#states)
1. [Services](#services)
1. [Providers](#providers)
1. [Configurations](#configurations)
1. [Helpers](#helpers)
1. [Routes](#routes)
1. [Handlers](#handlers)
1. [Styles](#styles)

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


**[Back to top](#table-of-contents)**
  

Views Models ViewControllers ServiceControllers

Component: directiva de entitat
Decorator: directiva d’atribut que canvia comportament sense scope
Model:
Factory: creador de models amb nom (casi com state)
State / Collection: guarda collections i te $update
Service: fa IO
Support: a directive amb dom...
Helper: functions reusables
Tool: no angular