
# Vue.js

Vue is extremely similar to angularjs, with the added benefit that it is not end-of-life.

## Install CLI

```text
npm install -g @vue/cli
```

## Create and run a new project

```text
vue create hello-world
npm run serve
```

## Create production build

```text
npm run build
```

## Single file components

As you can see, the project which was created contains single file components, named .vue files.

Here's a skeleton one:

```javascript
<template>
    <p>Hello</p>
</template>

<script>
    export default {
        name: "AnotherComponent"
    }
</script>

<style scoped>

</style>
```

The anatomy of these files is:

* template - the HTML template 
* script - the controller code
* style - any CSS specific to this component

### Template

The template is the HTML view.  It uses an extremely angularjs like syntax.

Here's one showing a bunch of features.

```html
<template>
    <div>
        <p>This is Marc One</p>
        <p>What came in is {{somethingComingIn}}</p>
        <p>The message is {{message}}</p>
        <p>The message is also {{ message | marcsUpperCase }}</p>
        <p>Here's a currency {{ 123.34 | currency('£', 0) }}</p>
        <input type="text" v-model="nameFilter"/>
        <ul>
            <li v-bind:key="name" v-for="name in filteredNames">{{name}}</li>
        </ul>
        <input type="text" v-model="nameToAdd"/>
        <button v-on:click="addName">Add</button>
    </div>
</template>
```

### Script

The script is the controller.  Here's the controller for the view above.

```javascript
<script>
    export default {
        name: "MarcOne",
        props: {
            somethingComingIn: String
        },
        data: function() {
            return {
                message: "Geoff",
                names: [
                    'Julian',
                    'George',
                    'Anne'
                ],
                nameFilter: '',
                nameToAdd: ''
            }
        },
        methods: {
            addName: function() {
                this.names.push(this.nameToAdd);
                this.$emit("somethingGoingOut", this.nameToAdd)
                this.nameToAdd = '';
            }
        },
        computed: {
            filteredNames: function() {
                return this.names.filter( name => name.includes(this.nameFilter) );
            }
        },
        watch: {
            nameToAdd: function() {
                console.log("nameToAdd changed to: " + this.nameToAdd);
            }
        },
        mounted: function() {
            console.log('I have been mounted.  I could fetch some data.');
        },
        filters: {
            marcsUpperCase: function(input) {
                return input.toUpperCase();
            }
        }
    }
</script>
```

It doesn't seem very intuitive at first, but it is easy to use.  Here's what the parts are:

* props - this is a list of the incoming properties for the component
* data - this is the initial state.  It needs to be a function that returns the state.
* methods - this is where we put our methods
* computed - if feel like computing values based on the state, you can do it here.
* watch - use this if you want to fire a function when a state variable changes
* filters - you can define some formatting filters 
* mounted - this is one of many lifecycle hooks.  See https://vuejs.org/v2/guide/instance.html

Note that we're not using arrow functions on these functions.  We can't.  If you use them you find that "this" is not bound to the vue component.  (Because JavaScript is weird)

### Props

Props work as you would expect.  

We defined a prop in the child component.

```javascript
props: {
    somethingComingIn: String
},
```

In the parent template we can pass a value to the prop.

```html
<MarcOne something-coming-in="onion" />
```

You can see that it has munged the name of the prop from camelCase to kebab-case.  AngularJS did similar things.  That is an annoyance that React is free from.

### Events

For some reason, the way to raise callbacks from a child to a parent component isn't as intuitive as it is in React.

Here's the ugly part, this is what the emit call looks like in the child.

```javascript
this.$emit("somethingGoingOut", this.nameToAdd)
```

The rest is okay.  Here's the parent template assigning the event to a handler.

```html
    <MarcOne something-coming-in="onion" v-on:somethingGoingOut="iCaughtSomething" />
```

And here's the handler in the controller.

```javascript
methods: {
    iCaughtSomething: function(msg) {
        console.log("i caught: " + msg);
    }
```


### Adding a bunch of filters

Vue supports filters for formatting data, you can write your own, but doesn't include any by default. 

If you want a set of useful filters you can add them.

```text
npm install vue2-filters
```

In main.js add:

```javascript
import Vue2Filters from 'vue2-filters'

Vue.use(Vue2Filters)
```

Then use the filters in your template.

```html
<p>Here's a currency {{ 123.34 | currency('£', 0) }}</p>
```

# Vue-Router

A router lets us route URLs to components, so we can make a site with different pages, implemented in different components.

Install it:

```text
npm install vue-router
```

The original main.js looked like this.  It picks up the div#app from index.html renders the App component there.

```javascript
new Vue({
  render: h => h(App),
}).$mount('#app')

```

I couldn't get the tutorial to work, but this is what did work.

In main.js create routes and pass the router to Vue, but still render the App, like before:

```javascript
import VueRouter from 'vue-router'

Vue.use(VueRouter)

const routes = [
  { path: '/', component: AnotherComponent },
  { path: '/foo', component: MarcOne }
]

const router = new VueRouter({
  routes // short for `routes: routes`
})

new Vue({
  render: h => h(App),
  router,
}).$mount('#app')

```

Now in the App component, add some navigation and router-view into the template.  

The router-view element will be replaced by the current page.  (or rather, by the component that matches the current route)

```html
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <div>
      <ul>
        <li> <router-link to="/">Another</router-link> </li>
        <li> <router-link to="/foo">MarcOne</router-link> </li>
      </ul>
    </div>
    <hr/>
    <router-view/>
  </div>
</template>
```

The tutorials seem to suggest adding router-view into the index.html instead, and not rendering App.  That doesn't work at all.  I think Vue renders anything into div#app, it wipes out the original content there, so then we don't have any router-view.

Once again, I notice that React does this in a rather more elegent way than Vue.

# Vuex

Vuex provices centralised state management, it is extremely similar to Redux.  Instead of each component having its own state, there is a global state which has an API that the components can use.  This separates any data wrangling into components that have that purpose only.

Here's an example shows integrating the state store with a middleware API.  I'm always interested in that aspect.

https://github.com/vuejs/vuex/tree/dev/examples/shopping-cart

In that example, the way it is done is by adding "actions" to the reducer.  The actions call a separate module which calls the API.  When the API method completes, it calls commit with the result of the operation.  That message is then handled by the normal reducer and updates the store.

That's essentially the same as the pattern I saw in react-redux, but the example for that splits the API middleware away from the reducer entirely.   Rather than changing the original reducer to call the API, they used Redux's applyMiddleware function, which just adds the new action handlers into the global store.  That seems a bit tidier.

https://www.sohamkamani.com/blog/2016/06/05/redux-apis/

Here's another article going through differences between vuex and redux.

https://medium.com/javascript-in-plain-english/similarities-and-differences-between-vuex-and-redux-by-developing-an-application-be3df0164b22



