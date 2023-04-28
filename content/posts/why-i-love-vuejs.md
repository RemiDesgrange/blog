---
title: "Why I Love Vuejs"
date: 2020-04-11T09:30:54+02:00
draft: false
tags: ["JS", "Dev", "Opinions"]
category: ["JS", "Vue", "Dev"]
---

## Background

I'm a backend developer. I went to embedded software dev, that all the industry are now calling "IoT", because it is much more cooler, to SysAdmin, now called DevOps, to backend developper. I never learned frontend dev, I always use all the possible helper builds to help me not do it. 

But really don't like this way of using wrapper blindly and not understand what you do and copy paste StackOverflow stuff into my code. 


## Why Vue.js 

For those who don't know [Vue](https://vuejs.org/) please see the website. 

> Yeah but why not React ? or Angular ?

* React:
    * is not a framework, it's a library. React takes care of rendering. You need to add library for store, routing, etc... And there is now standard ways of doing it, and don't forget it is JavaScript, trend change every two weeks (length an Agile Spring, coincidence ?)
    * You don't have to use JSX (yeah another new stuff...) with react but everyone is doing it. I am sorry, but JSX is JS in HTML, to me it looks like the php in HTML. Yeah remember when symfony didn't exist yet and SQL was written directly in HTML next to `<table>` tag. Fun time, but never again sorry not sorry.
* Angular:
    * You need to write stuff in [TypeScript](https://www.typescriptlang.org/). That's another language to learn. I find the concept of typing js cool, but it definitely not flatten the learning curve. Funny fact, TypeScript comes from Microsoft, and Angular from Google.
    * You can integrate Angular in a page without npm, a bundler, etc... a ton of stuff. So if you have an existing website and you "just" want to add this framework to remove the burden of jQuery from this particular page (ex: a dahsboard page) it's extremely complex to do it with angular. Once you start with it, it will eat all the HTML of your project.


Vue for me is battery included, it comes alone, but have a [store](https://vuex.vuejs.org/), a [router](https://router.vuejs.org/), [SSR](https://ssr.vuejs.org/), etc... You just have to `npm install` it.

* Also doc is great 👍
* Community is awesome
* Stability: Vue 2 has been release in [2016](https://github.com/vuejs/vue/releases/tag/v2.0.0) and almost no breaking change in 4 years !
* You can starts small. 2 years ago I had this page on a company internal app. It was a dashboard. with jQuery it was a mess and adding new graph and map was a mess. So I just tried Vue on this page. How hard it was ? like this:

```html
<!-- ... redacted ...-->
<body>
<!-- shit pile of html-->
<script src="jQuery.js"></script>
<script src="myPageJSWithJqueryInIt.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
</body>
</html>
```

You can then use Vue. But will it spread to all the page ? Hell no !

```javascript
const app = new Vue({
  el: '#newVueElement',
})
```

```html
<!-- ... redacted ...-->
<body>
<!-- shit pile of html-->
<div id="newVueElement"></div>
<script src="jQuery.js"></script>
<script src="myPageJSWithJqueryInIt.js"></script>
<script src="https://cdn.jsdelivr.net/npm/vue"></script>
</body>
</html>
```

And once you have a reasonable amount of code written with Vue, you can start integrating `.vue` pages. For this you will need a much more advanced toolchain, but if you start having lots of components, it worst it. 

```vue
<template>
  <div class="message">
    {{ message }}
  </div>
</template>

<script>
export default {
  data() {
    return {
      message: "coucou"
    }
  }
}
</script>

<style>
.message {
  color: "green"
}
</style>
```

As you can see it is all perfectly separated. And this is maybe what I like the most !

I am also really excited about the upcoming [Vue3](https://github.com/vuejs/vue-next). Evan You the creator of Vue.js [gave a talk](https://www.youtube.com/watch?v=WLpLYhnGqPA) (maybe many) about it


