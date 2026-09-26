***Vue.js is an open-source JavaScript framework designed for building user interfaces (UIs) and [[Single-Page Applications (SPAs)]].***
***[Official documentation to learn from basic](https://vuejs.org/guide/quick-start.html)***

![[What is Vue.js]]

Vue is more like a structured JavaScript.

### Things you should know before starting:
Like with any framework, you should be comfortable with the underlying language first. In this case, it is JS:
- JavaScript Fundamentals
- Async Programming
- Array Methods
- Fetch APT / HTTP Requests
- NPM (Node Package Manager)

### Basic Layout of the Vue Component 
The basic vue script looks like this: [[Vue Components]]
Vue has something called an *[App](App%20in%20Vue)*, it is the main controller of your user interface. But before we do anything, we first need to create the Vue project, and the simplest way to do so is using [[Vue CDN]].
Each component in HTML or CSS has some properties and function that it fulfills via JS, this can be done easily via [[Data and States in Vue]]
After we create this controller, we attach the thing that this controller will control, i.e., [The HTML Script](Connecting%20Vue%20App%20with%20the%20HTML%20Script), 
The mount will attach all the things we want to be added in our page in the element where we have tried to mount our Vue App.
But how do we access the properties and functions of the object returned by `data()`? It is via `{{ }}`.
for example:
![[Pasted image 20250611213831.png]]
![[Pasted image 20250611214446.png]]

We could also do that thing in the `div` where we are mounting the app:
```HTML
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Random User Generator</title>
</head>
<body>
    <div id="app">
        <h1>
            Hellow World <br>This is {{firstName}} {{lastName}}'s practice project!!!
        </h1>
    </div>

    <script src="https://unpkg.com/vue@3/dist/vue.global.js"></script>
    <script src="app.js"></script>
</body>
</html>
```

But still we haven't seen how will we connect the HTML attributes with our Vue App, for that we use [[v-bind]].
Now with that, what if we want something to happen when user does something? Something like events in DOM, we have [events here in Vue](Events%20in%20Vue).

### CLI in Vue
#### What is the Vue CLI?
Vue CLI (Command Line Interface) is a command-line tool that helps you quickly set up, develop, and manage Vue.js applications. It provides a standard build setup and a suite of tools, so you can focus on building your app rather than configuring build tools from scratch.

---
#### How Does Vue CLI Work?
**1. Installation**
You install Vue CLI globally using npm (Node Package Manager):
```bash
npm install -g @vue/cli
```
This gives you access to the `vue` command in your terminal.

**2. Project Creation**
To scaffold (generate) a new Vue project, use:
```bash
vue create my-project
```
This command interactively guides you through setup options (like Babel, TypeScript, Vue Router, Vuex, ESLint, etc.), or lets you use the default preset for a quick start.

**3. Project Structure**
A Vue CLI-generated project includes:
- `public/`: Static files like `index.html`
- `src/`: Your source code, components, assets
- `package.json`: Project dependencies and scripts
- Config files for Babel, ESLint, etc.

**4. Development and Build**
Inside your project directory, you can run:
```bash
npm run serve
```
This starts a development server with hot-reload, so you see changes instantly in your browser.

For production builds:
```bash
npm run build
```
This compiles and minifies your app for deployment.

**5. Plugins and Extensibility**
Vue CLI supports a plugin architecture. You can add features (like TypeScript, PWA support, testing tools) via official or community plugins, either during project creation or later using:
```bash
vue add <plugin-name>
```
Plugins are npm packages, and their names start with `@vue/cli-plugin-` (official) or `vue-cli-plugin-` (community).

**6. Graphical User Interface (GUI)**
You can launch a web-based GUI to create and manage projects with:
````bash
vue ui
````
This opens a dashboard in your browser for project setup, dependency management, running tasks, and more.

---
#### Vue CLI Components

|Component|Description|
|---|---|
|CLI (`@vue/cli`)|Globally installed npm package providing the `vue` command for project scaffolding and tools|
|CLI Service|Local dependency (`@vue/cli-service`) handling build, serve, and configuration|
|CLI Plugins|Add-on npm packages to extend functionality (e.g., Babel, TypeScript, Router, Vuex)|
|CLI UI|Graphical interface for project creation and management|

---

Now lets have a look at [[Basic structure of a vue project]]

Now all this might seem overwhelming and there might be many doubts like how is that file used? Like for example, I create a project, now it has `App.vue`, and `main.js`, and stuff, but there is no script tag in index.html, and `package.json` does not mention any starting point, so how is it all working?

When you create a Vue project (using Vue CLI or Vite), it may seem mysterious how your code runs—especially since `index.html` in the `public/` folder doesn’t directly include a `<script>` tag for your JavaScript, and `package.json` doesn’t specify a direct entry point. Here’s how it all works under the hood:

#### How Vue Project Bootstrapping Works

##### **1. The Role of `index.html`**
- The `public/index.html` file is a template, not the actual HTML file served in development or production.
- It contains a `<div id="app"></div>`, which is where your Vue app will be mounted.
- You’ll notice a comment like `<!-- built files will be auto injected -->`. This means that during the build process, the actual JavaScript and CSS files generated from your source code will be automatically injected into this HTML file.

##### **2. The Build Tool (Webpack/Vite) Handles Injection**
- When you run `npm run serve` (development) or `npm run build` (production), a build tool (like Webpack, via Vue CLI, or Vite) processes your source files.
- The tool bundles your code (including `main.js`, `App.vue`, and all dependencies) into one or more JavaScript files.
- It then injects `<script>` tags referencing these built files into the final HTML that gets served.
- In development, this happens in memory; in production, the built files appear in the `dist/` folder.

##### **3. The Role of `main.js`**
- `main.js` is the true entry point for your Vue application.
- It imports Vue, your root component (`App.vue`), and other dependencies like the router or store.
- It creates the Vue app instance and mounts it to the `#app` element in the HTML.
- Example:
    ```js
    import { createApp } from 'vue'; import App from './App.vue'; createApp(App).mount('#app');
	```
- This code tells Vue: “Render the `App.vue` component inside the element with id `app` in the HTML.”

##### **4. The Role of `App.vue` and Components**
- `App.vue` is the root component. It defines the main layout and structure of your application.
- All other components are imported and used inside `App.vue` or its child components.
- Components are imported and exported using ES6 module syntax.
- When you create a new component, you import it where needed and register it, so Vue knows how to render it.

##### **5. Why No Script Tag in `index.html` or Entry in `package.json`?**
- The build tool (Webpack or Vite) is configured (often via hidden or default settings) to use `main.js` as the starting point.
- It handles all the bundling and injection automatically, so you don’t have to manually edit `index.html` or specify an entry point in `package.json`
- `package.json` does specify scripts like `"serve"` or `"build"`, which trigger the build tool to start the process

#### The .vue file and it's use:
A `.vue` file is called a Single-File Component (SFC) in Vue.js. It’s a custom file format that allows you to define a Vue component’s structure, logic, and styling all in one file, using an HTML-like syntax

---
##### Structure of a `.vue` File
A typical `.vue` file contains up to three main sections:
- `<template>`: Contains the HTML markup for the component. This is what gets rendered in the DOM.
- `<script>`: Contains the JavaScript code, including the component’s logic, data, methods, props, and lifecycle hooks. This section exports the component definition.
- `<style>`: Contains CSS (optionally scoped to the component) for styling.

Example:
```vue
<template>
  <button @click="count++">You clicked me {{ count }} times.</button>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    }
  }
}
</script>

<style scoped>
button {
  color: blue;
}
</style>
```
- **`<template>`**: Defines the button and how it displays the `count`.
- **`<script>`**: Handles the logic for incrementing the count.
- **`<style scoped>`**: Styles only this button, not others on the page.

Here, in `<script>`, we know what `export` does, and what `{}` is, but what is `default` keyword?

The `default` keyword in `export default { ... }` is part of JavaScript's ES module system. It designates a "default export" from a module, meaning the exported value (in your example, an object literal) is the primary thing this file offers to other modules.

**What does `default` mean here?**
- The `default` keyword marks this export as the module's main export, as opposed to "named" exports, which must be imported by their exact name.
- You can only have one default export per module, but you can have multiple named exports.
- When you import a default export, you can give it any name you like:
```JS
// In MyComponent.js
export default {
  // component definition
}

// In another file
import AnythingYouWant from './MyComponent'
```

Here, `AnythingYouWant` will receive the object exported as default, regardless of its original name

---
##### How Does a `.vue` File Work?
- **Not Standard HTML/JS/CSS**: Browsers can’t use `.vue` files directly. Instead, a build tool (like Vite or Vue CLI with Webpack) processes these files.
- **Build Step**: The build tool parses each section, compiles the template into render functions, processes the JavaScript, and extracts the CSS. It then bundles everything into standard JavaScript and CSS files that browsers can use.
- **Component Registration**: The default export from the `<script>` section is the component object. You import this in other components or in your app’s entry point (`main.js`) to use it.

#### What are components in Vue? What are custom components and how do they work?
##### 1. What are Vue Custom Components?
Vue custom components are reusable, self-contained building blocks that let you break your UI into independent pieces. Each component manages its own structure (template), logic (script), and style, making your code modular and maintainable.
**In Vue, a component is typically defined in a `.vue` file with:**
- `<template>`: The HTML structure.
- `<script>`: The JavaScript logic (exporting a component object).    
- `<style>`: CSS for the component.
##### 2. How Are Vue Components Used?
Consider:
![[Pasted image 20250612202722.png]]
**In your screenshot:**
- You import `HeaderComponent` from `Header.vue`.
- Register it in the `components` section of the default export in `App.vue`.
- Use `<HeaderComponent title="Hello!!!" />` in your template
**How it works:**
- When Vue renders your app, it sees the `<HeaderComponent />` tag in your template.
- Because you registered `HeaderComponent`, Vue knows to render the imported component in its place.
- Props (like `title`) are passed to the component, making it dynamic and reusable

**The build tool (Vite, Vue CLI) compiles all `.vue` files into JavaScript modules. During runtime, Vue handles the rendering and updating of these components in the DOM**

##### 3. What Happens Under the Hood?
- **Import:**  
    `import HeaderComponent from "./components/Header.vue";` brings in the component definition.
- **Register:**  
    `components: { HeaderComponent }` tells Vue that `<HeaderComponent />` in the template refers to this import.    
- **Render:**  
    When the app runs, Vue replaces `<HeaderComponent />` with the actual rendered output of the `Header.vue` component.

**This allows you to create your own HTML-like tags that encapsulate logic and style, just like native HTML elements but with your own functionality**

##### Vue Component's pre-defined keywords:
Here are some of the most common **pre-defined options (properties)** you can use in a Vue component definition (the object you export in a `.vue` file or pass to `createApp`). These options are recognized by Vue and have special meanings—they are not arbitrary keys, but part of Vue's API, just like `name` in our given screenshot.

In our screenshot it would cause an error if we used that component as the `name` requires it's value to be multi word via Pascal Case, i.e., like `HeaderCustom`.

| Option         | Purpose                                                                                    | Example Usage                                    |
| -------------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------ |
| **name**       | Internal name for the component (for debugging, recursion, DevTools, `<keep-alive>`, etc.) | `name: 'AppHeader'`                              |
| **data**       | Function returning the component’s reactive state object                                   | `data() { return { count: 0 } }`                 |
| **props**      | Declare properties (props) the component can receive from its parent                       | `props: ['title', 'user']`                       |
| **methods**    | Define functions/methods available to the component                                        | `methods: { increment() { ... } }`               |
| **computed**   | Define computed properties (reactive, cached based on dependencies)                        | `computed: { double() { return this.count*2 } }` |
| **components** | Register child components locally for use in this component's template                     | `components: { HeaderComponent }`                |
| **emits**      | Declare custom events this component can emit                                              | `emits: ['update', 'submit']`                    |
| **watch**      | Watch for changes in data or props and run code in response                                | `watch: { count(newVal, oldVal) { ... } }`       |
| **mounted**    | Lifecycle hook: called after the component is mounted                                      | `mounted() { ... }`                              |
| **created**    | Lifecycle hook: called after the component is created                                      | `created() { ... }`                              |
| **template**   | Inline template string (rarely used in `.vue` files)                                       | `template: '<div>Hello</div>'`                   |
| **mixins**     | Include reusable logic from mixin objects                                                  | `mixins: [myMixin]`                              |
| **setup**      | (Vue 3 Composition API) Setup function for composition API logic                           | `setup(props) { ... }`                           |
| **slots**      | (Implicit) Used for content distribution via `<slot>` tags                                 | `<slot></slot>` in template                      |

##### Properties(props) in Vue Components:
**Props** (short for "properties") are a fundamental feature in Vue that allow you to pass data from a parent component(i.e., the script where we are using the child component, like `App.vue` in our screenshot) to a child component(the component that we are using itself). 

They enable reusable, dynamic components by letting the parent specify values the child should use.

###### How Props Work
1. **Parent Component:**  
    Passes data to a child component using custom attributes in the template.
    ```vue
    <!-- App.vue --> 
    <HeaderComponent title="Hello!!!" />
	```
    Here, `title="Hello!!!"` is a prop being passed to `HeaderComponent`.
    
2. **Child Component:**  
    Declares which props it expects to receive using the `props` option.
    ```    js
    // Header.vue 
    export default {   
	    props: ['title'] 
	}
	```
    Now, inside `HeaderComponent`, you can use `title` just like a data property.

###### Declaring Props
You can declare props in two main ways:
1. **Array Syntax**
```js
export default {   props: ['title', 'subtitle'] }
```
Use this for simple prop names (all are of type `any`).

2. **Object Syntax (Recommended for Validation)**
```js
export default {
  props: {
    title: String,           // expects a string
    likes: Number,           // expects a number
    isFavorite: Boolean,     // expects a boolean
    description: {
      type: String,
      required: true,        // must be provided
      default: 'N/A',        // default value if not provided
    }
  }
}
```
This syntax allows you to specify type, whether the prop is required, and default values.

###### Using props in the template of the component's Vue file
Once declared, you can use props in your component’s template as if they were data properties:
```Vue
<template>
  <h1>{{ title }}</h1>
</template>
```

###### Using props inside the parent like it is an attribute
You can pass as many props as you want:
```Vue
<HeaderComponent title="Hello!!!" subtitle="Welcome" :likes="10" :is-favorite="true" />
```

for example:
This is the child (`Header.vue`):
![[Pasted image 20250612204705.png]]
This is the parent(`App.vue`):
![[Pasted image 20250612204817.png]]
And the result:
![[Pasted image 20250612204835.png]]

### [[Vue Directives]]
Now that we are familiar with the basics of vue, time to look at vue components, built-in keywords that can be used to do stuff.
