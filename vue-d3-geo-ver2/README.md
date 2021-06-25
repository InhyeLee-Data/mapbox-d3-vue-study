# vue-d3-geo (2)
It is a project, following Alec Barrett's MSDV Major Studio 2 Example
- Using d3-geo and mapbox together on the same page of vue: https://github.com/anbnyc/major-studio-2-sp21/tree/main/05-geo 

In this version, both d3-geo and mapbox components are existent out of the App.vue.
Parent-child communication is crucial in this process, because both child components have to receive data and data change from the parent. 
Reference: https://medium.com/js-dojo/component-communication-in-vue-js-ca8b591d7efa 

## Perks
You need to have .env file in your root directory, in order to use to the mapbox access token. 
Dependencies can be checked inside package.json file

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).
