# Vue.js best practises

These recommendations should give you assistance to use Vue.js in a progressive and future-orientated way. They are focused around the core of the official Vue.js dependencies, that are the [vue-router](https://github.com/vuejs/vue-router), [vuex](https://github.com/vuejs/vuex) and [vue](https://github.com/vuejs/vue) itself.

## Guidelines

The recommendations differ in their importance. Check the following table to get an overview.

Keyword | Description
------- | ------------------------------------------------------------------------------------
Shall   | Shall is used in a sense of must. Ignoring it can result into problems
Should  | Should has a higher weight than may. It is the recommend approach
May     | May is used in a sense of optional. It doesn't make a difference for the application

## Best practises

### Shall

#### You shall use getters/setters for your data properties

Mutating a property should be as explicit as possible. Violating it could lead to an unwanted property mutation if you need the property as read-only.

```javascript
computed: {
  property () {
    return this.$store.state.property
  }
}
```

You have two possibilities to improve that.

Use computed getters/setters:

```javascript
computed: {
  property: {
    get () {
      return this.$store.state.property
    },

    set (property) {
      this.$store.commit('UPDATE_PROPERTY', property)
    }
  }
}
```

Use Vuex [getters](https://vuex.vuejs.org/en/getters.html):

```javascript
getters: {
  property: state => {
    return state.property
  }
}
```

[Back to top](#vuejs-best-practises)

#### You shall use `Vue.set` and `Vue.delete` to mutate/delete properties/array keys

Since Vue.js [can't detect property additions or deletions](https://vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats), you sould always use `Vue.set` to add and `Vue.delete` to delete properties or array keys. This is especially necessary when setting or deleting an array index or an object property. Please note: This won't be necessary in future versions.

[Back to top](#vuejs-best-practises)

#### You shall use pure functions

In Vue.js, many click directives should be handlers. Handlers do a couple of things. They should call other functions instead of combining the whole logic, which can lead to unpure functions with unwanted side effects. It's pretty easy to overlook that. Given a common example:

```js
setIsVisibleModal (isVisibleModal) {
  this.isVisibleModal = isVisibleModal
  if (isVisibleModal === false) this.setIsVisibleModalButton(true)
}
```

This function does two things, even so the function name clearly coerces you to do only one thing. It's easier and more testable to use a handler instead, like showed here:

```js
modalHandler (isVisibleModal) {
  this.setIsVisibleModal(isVisibleModal)
  if (isVisibleModal === false) this.setIsVisibleModalButton(true)
}
```

[Back to top](#vuejs-best-practises)

### Should

#### You should handle HTTP requests at Vuex actions only

HTTP requests should be independent from components. The underlying protocol, URI or route may change and you have to rewrite code at multiple places. It is better to have API-related operations at a centralized place instead of doing HTTP requests at the component level.

[Back to top](#vuejs-best-practises)

#### You should use component lazy loading

When you import a component the usual way, it gets loaded upfront, regardless of if it's needed. You can prevent that behavior with a function.

```javascript
const Component = () => import('./Component')
```

[Back to top](#vuejs-best-practises)

#### You should always define methods instead of cluttering hooks

If you need to setup up something during one of the hooks upfront, you should define a method instead of writing everything into the hook. With that, you can easily reinvoke the method later if necessary.

[Back to top](#vuejs-best-practises)

#### You should use Vuex strict mode

The Vuex is the centralized state of your application. To reason about state mutations, you must be aware of changes in an exact manner. When the strict mode is enabled and the state is mutated outside of a mutation handler, an error will be thrown.

[Back to top](#vuejs-best-practises)

#### You should not use `Vue.parent`

In general, components should be loosely coupled. However, there are situations where components can also be tightly coupled. If you have a dedicated mother-child-relationship, it is perfectly okay to use `Vue.parent` to access the mother component from the child. Nevertheless, if components can stand on their own, you shouldn't use it. It could be that the relationship get's interrupted and the child component is wrapped around another component which makes `Vue.parent` useless.

[Back to top](#vuejs-best-practises)

#### You should not use watchers `deep: true`

Using `deep: true` at a watcher leads to heavy calculations because Vue.js has to recursively check for property changes. It's better to use an explicit getter instead and watch for it.

[Back to top](#vuejs-best-practises)

#### You should follow an entity based workflow

Using entities as component names can heavily simplify reasoning about your application. A component is responsible for doing one thing and the name should reflect that. Therefore, you may end up using names like `UserGet`, `ProductPatch` or `ProductPost`. For common logic use mixins. Furthermore, your states should be an image of your server API. Try avoiding abstraction of your API.

[Back to top](#vuejs-best-practises)

#### You should let mother components handling errors

Consider a product page with a container component holding product components with another subcomponent loading product images. Thus, the responsibility chain goes down from the container component to the product component to the product images component. If there is an error with the product images, let the product component handle it. If the product component crashes, let the container decide what to do. This guarantees that the component with the most knowledge makes decisions.

[Back to top](#vuejs-best-practises)

### May

#### You may use a state constructor

You might find yourself reseting a Vuex state. Instead of setting verbosely every property of the state, define a state constructor instead. To do so, wrap the state into an arrow function and use `Object.assign` to reset the state.

```javascript
const stateConstructor = () => ({
  entity: ''
})
```

Mutations:

```javascript
RESET_ENTITY (state) {
  Object.assign(state, stateConstructor())
}
```

[Back to top](#vuejs-best-practises)

#### You may not always use state managament for a component

Most times, a component is a seperated, isolated unit of your application. Therefore, there is no need for such a component to be accessible from the outside or the other way around. You can save a lot of state managament if you ask yourself some questions:

- Do I need the components data or state elsewhere?
- Does an other component must mutate the components behaviour?
- Do I need the component elsewhere?

You may end up putting parts of your component into the store and the rest into the component.

[Back to top](#vuejs-best-practises)

#### You may use upper-case module names

To distinguish between modules and entities, you may use an upper-case module name. If you have many nested modules, this prevents you from mutating the root state, rather than a dedicated object for the entity. To use an upper-case namespace, simply define your modules like this:

```js
new Vuex.Store({
  modules: {
    User: user,
    Product: product,
    Basket: basket
  }
})
```

[Back to top](#vuejs-best-practises)

#### You may follow these naming conventions

Vue.js provides several varying APIs for DOM and state mutations or event communications. Since you decide about the interface names (like methods, events or DOM elements), a consistent naming convention is recommended. 

##### Components

| Entity              | Example            |
|---------------------|--------------------|
| Component file name | `SearchBar`        |
| Component           | `<search-bar/>`    |
| Events              | `makeUserVisible`  |
| Mixin file name     | `SearchBarMixin`   |
| Mixin               | `SearchBarMixin`   |
| Props               | `:is-red="true"`   |
| References          | `user-post`        |

##### Router

| Entity      | Example          |
|-------------|------------------|
| Route name  | `DashboardIndex` |

##### Vuex

| Entity             | Example          |
|--------------------|------------------|
| Module file name   | `user.module`    |
| Module             | `User`           |
| Mutation file name | `user.mutations` |
| Mutation functions | `SET_USER`       |
| Action file name   | `user.actions`   |
| Action functions   | `postUser`       |
| State file name    | `user.state`     |
| State properties   | `isUserVisible`  |

[Back to top](#vuejs-best-practises)
