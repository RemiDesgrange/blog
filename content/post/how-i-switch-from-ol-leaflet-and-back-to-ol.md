---
title: "How I Switch From OpenLayers to Leaflet and Back to OpenLayers"
date: 2019-12-05T18:54:28+01:00
draft: true
---

I'm coding [this](https://github.com/RemiDesgrange/nivo) side project right now about visualizing snow data and weather forecast.

It uses a map on every pages to navigate through different massifs, points, and select data.

I start using OL (OpenLayers) because we are heavy users at [my company](https://www.camptocamp.com). I'm more of a backend dev, so I do not know map library well. I briefly used [Leaflet](https://leafletjs.com/), [MapboxGL JS](https://github.com/mapbox/mapbox-gl-js), and I wanted to try this library. 

The project frontend is done in [Nuxt](https://nuxtjs.org/) a [Vue](https://vuejs.org/) framework that I am using in universal mode. It means that it's not simply a SPA, it has a node backend that can pre-generate the pages. It may be overkill for this project but I wanted to try how it works, and with Nuxt, passing from universal to spa is just a configuration.

So in order to integrate Vue with OpenLayers I used [Vuelayers]() which provide components and helpers to make OL more pleasant to use with Vue. 

## First approach to OL : too complex

I "simply" [^1] wanted to :

* display a bunch of point and polygon + 3/4 base layers;
* being able to have action on mouseover and click on those points/polygons;
* basic styling of these features (features=points/polygons)

[^1] we will get to "simply" later.

That is all ! And I find OL too complex because I needed to write too much code. I also did not understand all the concept of OL. It is a complex library and the doc is not that great. 

For example I tried to use multiple `ol/interaction/Select` but it was not possible until v6 (which was released a few weeks ago.) and it was a bit difficult at first (still is) to interact nicely and simply with the store ([vuex](https://vuex.vuejs.org))

OL doesn't even have a layer switcher you have to write it on your own !

So I decide to try Leaflet since OL seams too complex and too much needed to be written.

## 3h with leaflet : from "Waouh effect" to "Seriously?!"

So I did a `npm remove --save openlayers` and a `npm install --save leaflet` [^2]. I commented out the map component, and start using geojson and the standard map with some control (layer switcher). 

First I had to fight against nuxt, because leaflet try to do a lot of stuff with `window` even for component where you don't expect (like tooltips). Then you will find yourself writing html in the js because tooltips cannot be a component of it's own. 

It worked pretty fast, but I was stuck with the minimal stuff leaflet is capable of and with the bad design of it. Every stuff I wanted to  do end up with

> hum, could be much easier in openlayers, why do I need to reimplement that ?

[^2]: ok real call was `npm r vuelayers && npm i vue2leaflet`.

## Back to OL

Now time to `git reset --head HEAD` ☹.

