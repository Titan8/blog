+++
draft = false
date = "2016-10-31T08:42:14+08:00"
title = "Introduction to Vue"

+++
## Basic
```html
<div id="app">
  {{ message }}
</div>
```

```javascript
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```
Vue inherits a lot from angular. Both has concept of `directives` and `components`.

We will skip those similarities, but focus on differences here.

### Directive has Modifier and Arguments
```html
<input v-model.number="age" type="number">
<a v-bind:href="url"></a>
```
* Modifiers are special postfixes denoted by a dot, which indicate that a directive should be bound in some special way.
* Some directives can take an “argument”, denoted by a colon after the directive name.

### Component has slots
![slot](http://p1.bpimg.com/567571/d8c7009a5bc6f77a.jpg)

The API for a Vue component comes in three parts - props, events, and slots:

1. Props allow the external environment to pass data into the component
2. Events allow the component to trigger side effects in the external environment
3. Slots allow the external environment to compose the component with extra content.

```html
<my-component
  v-bind:foo="baz"
  v-bind:bar="qux"
  v-on:event-a="doThis"
  v-on:event-b="doThat"
>
  <img slot="icon" src="...">
  <p slot="main-text">Hello!</p>
</my-component>
```

## Vue Ecosystem

  * [vue-router](http://router.vuejs.org/) is officially-supported

  * [vuex](http://vuex.vuejs.org/) is a state management pattern + library for Vue.js applications.


## Comparison
| Metric        | Angular       | Angular 2 | React     | Vue       |
| ------------- |:-------------:|:---------:|:---------:| ---------:|
| Star(Github)  | 53015         | 17748     | 52844     | 32113     |
| Fork(Github)  | 26177         | 4525      | 9306      | 3645      |
| Issue(on/off) | 674/7460      | 811/6964  | 484/3336  | 66/3069   |
| Size          | 156K          | 636K      | 150K      | 64K       |
| LOC(TodoMVC)  |  442          | 198       | 520       | 221       |
| Size(TodoMVC) | 1.1M          | 1.6M      | 1.2M      | 294K      |
*created on 10/31/2016*


**PS:**

* Angular 2 requires Typescript while others require Javascript.
* Angular 2, React and Vue support cross-platform programming.
* Angular offer more features. But React and Vue get enhancement from
  additional libraries, which is authored and supported by community.
* From the metric above, Vue has the smallest community.

[Read more](http://vuejs.org/guide/comparison.html) on differences among these front-end Javascript frameworks.

You can use [vue-cli](https://github.com/vuejs/vue-cli) to scaffold Vue.js projects.
