React Native is a framework that lets you build mobile apps for both iOS and Android using JavaScript, the same language used for websites.
It makes developing an app OS free, i.e., no matter Android or IOS, it can run that app via a single codebase.


# Theory

## Understanding How Mobile Apps Display Content

Mobile apps work through three main components:
- the user interface (what you see on screen), 
- the code (the logic that makes things happen), and 
- data storage. 

When you tap a button on your phone, sensors detect the touch and send a message to the operating system (iOS for iPhones, Android for others). The operating system then tells your app's code to respond, which might update what's displayed, save information, or trigger other actions.

React Native uses **native components** that map directly to actual iOS and Android views. For example, when you write `<Text>` in React Native, it becomes a `TextView` on Android and a `UITextView` on iOS—these are the real building blocks each platform uses to display text.

So how are they different from websites?
How do websites work then?

## How Websites Work in Browsers

Websites work completely differently from native apps. When you visit a website, your browser downloads HTML (structure), CSS (styling), and JavaScript (interactivity) files. The browser's rendering engine then:
​
- Parses the HTML into a DOM (Document Object Model) tree—a structured representation of the page

- Processes CSS to create styling rules (CSSOM)

- Combines these to calculate where every element should appear and how big it should be   

- Finally paints pixels to your screen using the GPU​

This happens every time you load a webpage, whereas native apps have pre-compiled code that runs directly on your phone's operating system.


# The CLI
React Native development requires installing several components: 
	Node.js, 
	Java Development Kit (JDK), 
	Android SDK, and 
	a command-line interface (CLI). 

You have two main CLI options:

- **Expo CLI** (beginner-friendly) or 
	- **Expo CLI** is a wrapper framework that simplifies React Native development by removing most configuration complexity.
	
- **React Native CLI** (full control)
	- **React Native CLI** is the traditional bare-bones approach that gives you complete control over your project.


# Basic React Native File Structure

A simple React Native file looks like this:

```javascript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

export default function App() {
  return (
    <View style={styles.container}>
      <Text style={styles.text}>Hello World!</Text>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
  text: {
    fontSize: 20,
    color: 'blue',
  },
});
```

**Three essential parts:**

1. **Import statements** - brings in React and components from `react-native`
2. **Component function** - returns JSX (looks like HTML but isn't). JSX stands for **JavaScript XML** (or JavaScript Syntax Extension), and it lets you write HTML-like code directly inside JavaScript files. It's a syntax extension that makes building user interfaces easier by allowing markup and logic to live together.
3. **StyleSheet** - defines your styles using JavaScript objects

Now, we write in `.tsx` files in React-Native. The **`.tsx`** file extension stands for **TypeScript JSX**.

It is a specific file type used in web development that combines **TypeScript** (a typed superset of JavaScript) with **JSX** (JavaScript XML), which is syntax primarily used by the React framework to write HTML-like code within JavaScript.

Here is a breakdown of why this specific extension exists and how it differs from others:

### 1. The Core Combination

- **TypeScript (`.ts`):** Standard TypeScript files allow you to write JavaScript with static types (strings, numbers, interfaces, etc.) to catch errors early. However, standard `.ts` files generally do not support JSX syntax.
	
- **JSX:** This syntax allows you to write tags like `<div>Hello</div>` directly in your code.
	
- **`.tsx`:** This extension tells the TypeScript compiler: _"This file contains TypeScript syntax, but also expect JSX tags, so please handle them correctly instead of throwing syntax errors."_

# JSX

## What is JSX?

JSX (JavaScript XML) is a **syntax extension for JavaScript** created by React. It's **not standard JavaScript**—it's React-specific and lets you write HTML-like code inside JavaScript files.
## How it works:

JSX code gets **transpiled (converted) to regular JavaScript** by tools like Babel before running in the browser. Browsers can't understand JSX directly.

**You write:**

```jsx
const element = <h1>Hello, world!</h1>;
```

**Babel converts to:**

```javascript
const element = React.createElement('h1', null, 'Hello, world!');
```

Every JSX tag becomes a `React.createElement()` function call.

## JSX Syntax Basics

### HTML-like tags in JavaScript:

```jsx
const greeting = <Text>Hello!</Text>;
```

### JavaScript expressions with `{}`:

```jsx
const name = "John";
const greeting = <Text>Hello, {name}!</Text>;  // Output: Hello, John!
```

**Single `{}`** = JavaScript expression inside JSX.[^11][^5]

### Objects with `{{}}`:

```jsx
<View style={{backgroundColor: 'blue', padding: 10}}>
```

- Outer `{}`: JavaScript mode
- Inner `{}`: Object literal `{backgroundColor: 'blue', padding: 10}`[^11]


### Components must return one parent element:

```jsx
// WRONG
return (
  <Text>Hello</Text>
  <Text>World</Text>
);

// CORRECT
return (
  <View>
    <Text>Hello</Text>
    <Text>World</Text>
  </View>
);
```


### Attributes:

- Use `className` instead of `class` (JavaScript reserved word)[^6][^5]
- Use camelCase: `backgroundColor`, `onClick`[^5]

```jsx
<View className="container" onClick={handleClick}>
```


### Self-closing tags need `/`:

```jsx
<Image source={{uri: 'pic.jpg'}} />
<TextInput />
```


### Embedding expressions:

```jsx
const age = 25;
const user = {name: "John"};

<View>
  <Text>{age + 5}</Text>           {/* 30 */}
  <Text>{user.name}</Text>          {/* John */}
  <Text>{age > 18 ? "Adult" : "Minor"}</Text>  {/* Adult */}
</View>
```


## Key differences from JavaScript:

| JavaScript | JSX |
| :-- | :-- |
| Standard language, runs anywhere | React-specific syntax extension [^2][^5] |
| Browsers understand it directly | Must be transpiled to JS first [^2][^3] |
| Create elements with `createElement()` | Write HTML-like syntax [^2][^5] |
| `.js` files | Usually `.jsx` files (but `.js` works too) [^1][^6] |

## Is JSX required?

**No.** You can write React without JSX using `React.createElement()` directly, but JSX makes code much cleaner and easier to read.

**Without JSX:**

```javascript
React.createElement('div', null, 
  React.createElement('h1', null, 'Hello'),
  React.createElement('p', null, 'World')
);
```

**With JSX:**

```jsx
<div>
  <h1>Hello</h1>
  <p>World</p>
</div>
```

# Starting out:

You are likely looking at a folder structure that seems deceptively simple compared to older tutorials. The latest version of React Native Expo (SDK 50+) has shifted entirely to a **file-based routing system** called **Expo Router**.

Instead of writing code to define your navigation structure, your **file structure** _is_ your navigation structure.

Here is a comprehensive breakdown of the starting point.

---

### 1. The Core: The `app` Directory

In the modern version, the heart of your application is the `app` folder. This is not just a place to store files; it is your router. Every file you create inside this folder automatically becomes a screen in your app.

- **How it works:** If you create a file named `settings.tsx` inside the `app` folder, you automatically get a route usable via `/settings`. You do not need to manually register this screen in a configuration file.
    
- **The benefit:** It mimics how the web works (like Next.js). You can organize screens by simply organizing files.
    

> **💡 Legacy Comparison:**
> 
> In older versions (React Navigation), you had to manually import every screen component into a central `App.js` file and pass them into a `<Stack.Navigator>` or `<Tab.Navigator>`. If you forgot to register a screen, you couldn't navigate to it.

---

### 2. The Entry Point: `app/index.tsx`

This is arguably the most important file to understand immediately. This file represents the **root screen** of your app—the first thing a user sees when they launch the application (correlating to the path `/`).

- **Function:** It exports a standard React Native component (usually a functional component).
    
- **Usage:** You build your UI here using `<View>`, `<Text>`, and other components, just like any other component.
    


```TypeScript

import { View, Text } from 'react-native';

export default function HomeScreen() {
  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'center' }}>
      <Text>Welcome to the App!</Text>
    </View>
  );
}
```

> **💡 Legacy Comparison:**
> 
> Historically, the entry point was a file named `App.js` located in the root directory. That file typically contained the entire logic for the app's setup, including providers and the root navigator. Now, `App.js` is gone; `app/index.tsx` is just the _content_ of the first page.

---

### 3. The Wrapper: `app/_layout.tsx`

If `index.tsx` is the _content_, then `_layout.tsx` is the _frame_. This file is special because it wraps around your screens. It persists while you navigate between different screens.

- **Role:** It defines the **Navigation UI** (like the header bar, tab bar, or a drawer) and global providers (like Theme Providers, Auth Providers, or Redux stores).
    
- **The `<Slot />` or `<Stack />`:** Inside this file, you will usually see a component exported from `expo-router` (like `<Stack />`). This component acts as a placeholder where your screens (like `index.tsx`) will be rendered.
    

Example of a Stack Layout (creates a native navigation stack with headers):


```TypeScript
import { Stack } from 'expo-router';

export default function Layout() {
  return (
    <Stack>
      {/* Configures the header for the index route */}
      <Stack.Screen name="index" options={{ title: 'Home' }} />
    </Stack>
  );
}
```

> **💡 Legacy Comparison:**
> 
> In the old version, this logic lived inside the `NavigationContainer` in `App.js`. You had to wrap your whole app in providers manually. In Expo Router, the `_layout` file handles this hierarchy automatically—anything in `_layout` applies to every screen inside that folder.

---

### 4. Navigation & Linking
Since routes are files, navigating is done using URLs, even though it renders native views.

- **The `<Link>` Component:** You use this to move between screens. It works similarly to an `<a>` tag in HTML.
- **Type Safety:** The latest Expo handles TypeScript automatically. If you try to link to `/profile` but the file `app/profile.tsx` does not exist, your editor will flag it as an error.
    


```TypeScript
import { Link } from 'expo-router';

// In your view
<Link href="/settings">Go to Settings</Link>
```

> **💡 Legacy Comparison:**
> 
> Previously, you used a hook `const navigation = useNavigation();` and called functions like `navigation.navigate('Settings')`. While you can still use the imperative router object (`router.push('/settings')`), the declarative `<Link>` component is now the preferred standard for simple UI navigation.

---

### 5. Dynamic Routes (`[id].tsx`)

Modern Expo handles "parameterized" screens (like a user profile `user/123`) using square brackets in filenames.

- **How it works:** If you name a file `app/user/[id].tsx`, it will match any route like `/user/1` or `/user/abc`.
    
- **Accessing Data:** Inside that file, you use a hook `useLocalSearchParams()` to grab the `id` from the URL.
    

> **💡 Legacy Comparison:**
> 
> You used to have to define routes with generic parameters like `Stack.Screen name="User" component={User} initialParams={{ id: 42 }}`. You then extracted params via `route.params`. The new file-naming convention replaces that configuration entirely.

---

### 6. Configuration: `app.json`

Located in the root (outside `app`), this file controls the "metadata" of your native app. This is where you configure things that aren't code, but _properties_ of the app bundle.

- **Key Settings:**
    
    - **slug/name:** The internal name and display name of your app.
        
    - **splash:** Settings for the launch screen image.
        
    - **ios/android:** Specific build settings like bundle identifiers (`com.yourname.app`) and permissions.
        

> **💡 Legacy Comparison:**
> 
> This file has remained largely consistent. However, in the "Old Architecture" (bare React Native CLI), these settings were scattered across native files like `Info.plist` (iOS) and `AndroidManifest.xml` (Android). Expo abstracts these into this single JSON file.

### Summary Checklist for the "New" Start

1. **Open `app/_layout.tsx`** to see _how_ your app navigates (Stack, Tabs, or Drawer).
2. **Open `app/index.tsx`** to edit the first screen the user sees.
3. **Create a new file** in `app/` to create a new screen.

# Components in ReactNative

Components are **reusable building blocks** that make up your React Native app's user interface. They're JavaScript functions or classes that return JSX describing what should appear on screen.

## Two Types of Components

- **Functional Components (modern approach)** - Simple JavaScript functions that take `props` and return JSX:

```javascript
const Greeting = (props) => {
  return <Text>Hello, {props.name}!</Text>;
};
```

- **Class Components (older style)** - ES6 classes that extend `React.Component` and have a `render()` method:[^9][^7]

```javascript
class Greeting extends Component {
  render() {
    return <Text>Hello, {this.props.name}!</Text>;
  }
}
```

Modern React Native development uses **functional components exclusively**. Class components still exist for backward compatibility but aren't used in new code.[^10][^9]

## Core Components vs Custom Components

**Core Components** are built-in components provided by *React Native*:

- `View` - Container for layout
- `Text` - Display text
- `Image` - Display images
- `TextInput` - User text input
- `ScrollView` - Scrollable container
- `FlatList` - Efficient list rendering
- `Button` - Basic button
- `TouchableOpacity` - Touchable wrapper with feedback
- `Modal` - Popup overlay
- 

**Custom Components** are components you create by combining core components:[^6][^8]

```javascript
const ProfileCard = (props) => {
  return (
    <View>
      <Image source={{uri: props.photo}} />
      <Text>{props.name}</Text>
      <Text>{props.bio}</Text>
    </View>
  );
};
```


## How Components Work Together

Components can be **nested** inside each other to build complex UIs:[^6]

```javascript
const App = () => {
  return (
    <View>
      <ProfileCard name="Alice" photo="url1" bio="Developer" />
      <ProfileCard name="Bob" photo="url2" bio="Designer" />
    </View>
  );
};
```


## Props - Passing Data

Components receive data through **props** (properties), which are passed like HTML attributes:

```javascript
const Car = (props) => {
  return <Text>I am a {props.color} car!</Text>;
};

// Usage
<Car color="red" />  // Outputs: "I am a red car!"
<Car color="blue" /> // Outputs: "I am a blue car!"
```

Props make components reusable—the same component displays different data based on what props you pass.



## Component Export/Import

Store components in separate files and import them where needed:[^6]

```javascript
// ProfileCard.js
export default function ProfileCard(props) {
  return <View><Text>{props.name}</Text></View>;
}

// App.js
import ProfileCard from './ProfileCard';

export default function App() {
  return <ProfileCard name="Alice" />;
}
```

Given your Java background, think of components like classes: you define them once, then create multiple instances with different properties (props). The main difference is components are functions that return UI elements instead of objects with methods.


# what is props.children in ReactNaitve?

`props.children` in React Native (works same as React) is a **special prop that contains whatever you put between the opening and closing tags** of a component.[^4][^7]

## How it works:

When you write this:

```javascript
<Card>
  <Text>Hello World</Text>
</Card>
```

```
React automatically passes `<Text>Hello World</Text>` to the `Card` component as `props.children`.
```


## Using props.children:

```javascript
function Card(props) {
  return (
    <View style={styles.card}>
      {props.children}
    </View>
  );
}
```

Or with destructuring:

```javascript
function Card({ children }) {
  return (
    <View style={styles.card}>
      {children}
    </View>
  );
}
```

Now you can wrap any content inside `Card`:

```javascript
<Card>
  <Text>Title</Text>
  <Image source={uri: 'pic.jpg'} />
  <Button title="Click" />
</Card>
```

All those elements get passed as `children` and rendered inside the `View`.[^10][^4]

## When to use it:

- **Wrapper components** like containers, cards, modals[^8][^5]
- **Layout components** where the content changes but the structure stays the same[^8]
- **Reusable components** that need flexibility in what they display[^7][^5]

If you don't render `{props.children}` inside the component, the nested content gets ignored

## [Styling](Styling%20in%20ReactNative)

React Native uses JavaScript objects for styling, not CSS. You have two options:

- **Inline styles:**

```javascript
<Text style={{fontSize: 16, color: 'red'}}>Text</Text>
```

- **StyleSheet (recommended):**

```javascript
const styles = StyleSheet.create({
  text: {
    fontSize: 16,
    color: 'red',
    margin: 10,
  }
});
```

StyleSheet provides better performance and type checking. *Style properties use camelCase* (`backgroundColor` not `background-color`) and numbers default to pixels.

## How Components Work

Components are functions that return JSX. 
They must:

- Import React
- Import components from `react-native`
- Return a single parent element (usually `<View>`)
- Export the function with `export default`

Example with button interaction:

```javascript
import React from 'react';
import { View, Text, Button, Alert } from 'react-native';

export default function App() {
  return (
    <View>
      <Text>Press the button</Text>
      <Button 
        title="Click me" 
        onPress={() => Alert.alert('Button pressed!')}
      />
    </View>
  );
}
```


# How it all connects:

## App.js is the Entry Point

`App.js` is the **main component** where your React Native app's user interface begins. It's the root container that holds all your other components—every screen, button, and UI element ultimately gets rendered through this file.[^3][^5]

## The App Function

The `App` function is a **React functional component**. In modern React, components are just JavaScript functions that return JSX describing what should appear on screen.[^4][^5]

Here's what happens:

```javascript
const App = () => {
  return (
    <View>
      <Text>Hello, React Native!</Text>
    </View>
  );
};

export default App;
```

**Breaking it down:**

**`const App = () => { ... }`** - This creates a function called `App` using arrow function syntax. It's a component, which is just a function that returns JSX.[^5]

**`return (...)`** - The function returns JSX that defines what displays on screen.[^5]

**`export default App`** - This exports the component so other files can import and use it. Specifically, `index.js` imports this to register it with React Native.[^5]

## How App.js Connects to Your Phone

The actual connection happens in `index.js`:[^6][^5]

```javascript
import { AppRegistry } from 'react-native';
import App from './App';

AppRegistry.registerComponent('YourAppName', () => App);
```

`AppRegistry.registerComponent()` tells React Native which component to render first when your app launches. It's the bridge between your JavaScript code and the native iOS/Android platform. Think of `index.js` as the "main method" in Java—it's the execution starting point.[^6][^5]

## Component Tree Structure

`App.js` serves as the **root of your component tree**. You build your entire app by nesting components inside `App`:[^3][^5]

```javascript
const App = () => {
  return (
    <View>
      <Header />
      <MainScreen />
      <Footer />
    </View>
  );
};
```

Given your Java background, think of `App` as your application's main class—everything else extends from here. Components are like objects that you instantiate and compose to build the full interface.


# Key Differences from Web

- No HTML tags - use `View` instead of `div`, `Text` instead of `p`[^3]
- All text *must* be inside `<Text>`
- Styles are JavaScript objects, not CSS files
- Flexbox is default layout (no need to specify `display: flex`)
- Import every component you use from `react-native`[^4]

Given your Java background, think of components like classes - you create instances (components) with properties (props) and internal state, then compose them to build your UI

# Components

# `KeyboardAvoidingView` in reactNative?

`KeyboardAvoidingView` is a component that **automatically adjusts its layout when the keyboard appears** so that text inputs don't get hidden behind it.[^1][^3]

## The problem it solves:

When you tap a `TextInput` near the bottom of the screen, the keyboard covers it and you can't see what you're typing. `KeyboardAvoidingView` fixes this by moving content up when the keyboard opens.[^5]

## Basic usage:

```javascript
import { KeyboardAvoidingView, TextInput, Platform } from 'react-native';

<KeyboardAvoidingView 
  behavior={Platform.OS === 'ios' ? 'padding' : 'height'}
  style={{ flex: 1 }}
>
  <TextInput placeholder="Type here..." />
</KeyboardAvoidingView>
```


## Key props:

**behavior** - How to adjust the view:

- `"padding"` - Add padding to the bottom
- `"height"` - Adjust the view's height
- `"position"` - Adjust the view's position
- `undefined` - Disabled (useful for Android where it's often not needed)[^9]

**keyboardVerticalOffset** - Extra space from the top (useful with headers):[^6][^1]

```javascript
<KeyboardAvoidingView 
  behavior="padding"
  keyboardVerticalOffset={Platform.OS === 'ios' ? 40 : 0}
>
```

**enabled** - Enable/disable the component:[^2][^1]

```javascript
<KeyboardAvoidingView enabled={true}>
```


# What is `StatusBar` component in `ReactNative`

`StatusBar` is a component that **controls the appearance of the device's status bar**—the top bar showing time, battery, Wi-Fi, and network icons.

## Basic usage:

```javascript
import { StatusBar } from 'react-native';

<StatusBar 
  backgroundColor="#2196F3"
  barStyle="light-content"
  hidden={false}
/>
```


## Key props:

**barStyle** - Controls icon/text color:[^4][^1]

- `"default"` - Platform default
- `"light-content"` - White text/icons (for dark backgrounds)
- `"dark-content"` - Dark text/icons (for light backgrounds)

**backgroundColor** - Status bar background color (Android only)[^1][^4]

**hidden** - Show/hide the status bar:[^2][^1]

```javascript
<StatusBar hidden={true} />  // Hides status bar
```

**animated** - Animate changes to status bar props[^4]

**translucent** - Make status bar translucent (Android)[^6]

**showHideTransition** - Animation type when showing/hiding (iOS):[^4]

- `"fade"`
- `"slide"`
- `"none"`


## Example:

```javascript
import React from 'react';
import { View, StatusBar } from 'react-native';

export default function App() {
  return (
    <View>
      <StatusBar
        backgroundColor="#61dafb"
        barStyle="dark-content"
        hidden={false}
        animated={true}
      />
      {/* Your app content */}
    </View>
  );
}
```


## Multiple StatusBars:

You can have multiple `StatusBar` components mounted at the same time—they merge in the order they're mounted. This is useful with navigation where different screens need different status bar styles.[^3][^2][^1]

The component doesn't render any visible UI—it just controls the native status bar appearance

# FlatList in React Native

`FlatList` is a **performant component for rendering scrollable lists** in React Native. It only renders items that are currently visible on screen, making it much more efficient than using `ScrollView` with `.map()` for large datasets.[^2][^5][^7]

## Basic Syntax

```javascript
<FlatList
  data={arrayOfItems}
  renderItem={({item}) => <YourComponent item={item} />}
  keyExtractor={item => item.id}
/>
```


## Essential Props

**`data`** (required) - The array of items you want to display:[^3][^2]

```javascript
const data = [
  {id: '1', title: 'First Item'},
  {id: '2', title: 'Second Item'},
  {id: '3', title: 'Third Item'},
];
```

**`renderItem`** (required) - Function that defines how each item displays:[^5][^3]

```javascript
const renderItem = ({item}) => (
  <Text style={styles.item}>{item.title}</Text>
);
```

**`keyExtractor`** - Extracts a unique key for each item. By default, FlatList looks for `key` or `id` properties:[^6][^3][^5]

```javascript
keyExtractor={item => item.id}
```


## Complete Example

```javascript
import { FlatList, Text, View, StyleSheet } from 'react-native';

const App = () => {
  const data = [
    {id: '1', title: 'Iced Espresso'},
    {id: '2', title: 'Cappuccino'},
    {id: '3', title: 'Latte'},
  ];

  const renderItem = ({item}) => (
    <View style={styles.item}>
      <Text>{item.title}</Text>
    </View>
  );

  return (
    <FlatList
      data={data}
      renderItem={renderItem}
      keyExtractor={item => item.id}
    />
  );
};

const styles = StyleSheet.create({
  item: {
    padding: 20,
    borderBottomWidth: 1,
  },
});
```

This renders a simple scrollable list of coffee drinks.[^5]

## Useful Optional Props

**`horizontal`** - Makes the list scroll horizontally instead of vertically:[^6]

```javascript
<FlatList horizontal data={data} renderItem={renderItem} />
```

**`ListHeaderComponent`** - Adds content at the top of the list:[^5][^6]

```javascript
<FlatList
  ListHeaderComponent={() => <Text>Menu</Text>}
  data={data}
  renderItem={renderItem}
/>
```

**`ListFooterComponent`** - Adds content at the bottom:[^6][^5]

```javascript
<FlatList
  ListFooterComponent={() => <Text>End of list</Text>}
  data={data}
  renderItem={renderItem}
/>
```

**`ListEmptyComponent`** - Displays when the data array is empty:[^5]

```javascript
<FlatList
  ListEmptyComponent={() => <Text>No items found</Text>}
  data={[]}
  renderItem={renderItem}
/>
```

**`ItemSeparatorComponent`** - Adds separators between items:[^3]

```javascript
<FlatList
  ItemSeparatorComponent={() => <View style={{height: 1, backgroundColor: 'gray'}} />}
  data={data}
  renderItem={renderItem}
/>
```

# Input component in react-native-element

`Input` from `react-native-elements` is an **enhanced text input component** with built-in styling, icons, labels, and error messages. It wraps React Native's `TextInput` with additional features.[^1][^3]

## Installation \& Import:

```javascript
import { Input } from 'react-native-elements';
```


## Basic usage:

```javascript
<Input
  placeholder='Enter your name'
/>
```


## Key props:

**placeholder** - Placeholder text[^1]

**value \& onChangeText** - Control input value:[^1]

```javascript
const [text, setText] = useState('');

<Input
  value={text}
  onChangeText={value => setText(value)}
/>
```

**label** - Display label above input:[^1]

```javascript
<Input
  label="Username"
  placeholder="Enter username"
/>
```

**leftIcon \& rightIcon** - Add icons:[^1]

```javascript
<Input
  placeholder='Email'
  leftIcon={{ type: 'font-awesome', name: 'envelope' }}
/>
```

**errorMessage** - Show error text below input:[^1]

```javascript
<Input
  placeholder='Password'
  errorMessage='Password is required'
  errorStyle={{ color: 'red' }}
/>
```

**secureTextEntry** - Hide text for passwords:[^1]

```javascript
<Input
  placeholder="Password"
  secureTextEntry={true}
/>
```

**disabled** - Disable input[^1]

## Styling props:

- `containerStyle` - Outer container style[^1]
- `inputContainerStyle` - Input container style[^1]
- `inputStyle` - Text input style[^1]
- `labelStyle` - Label text style[^1]


## Methods:

Access methods using refs:[^3][^1]

```javascript
const inputRef = React.createRef();

<Input ref={inputRef} />

// Call methods:
inputRef.current.focus();      // Focus input
inputRef.current.blur();       // Remove focus
inputRef.current.clear();      // Clear text
inputRef.current.shake();      // Shake animation for errors
```


## Complete example:

```javascript
import React, { useState } from 'react';
import { Input } from 'react-native-elements';

const LoginForm = () => {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [error, setError] = useState('');

  return (
    <>
      <Input
        placeholder='Email'
        leftIcon={{ type: 'font-awesome', name: 'envelope' }}
        value={email}
        onChangeText={setEmail}
      />
      <Input
        placeholder='Password'
        leftIcon={{ type: 'font-awesome', name: 'lock' }}
        secureTextEntry={true}
        value={password}
        onChangeText={setPassword}
        errorMessage={error}
      />
    </>
  );
};
```

The `Input` component inherits all standard `TextInput` props, so you can use any React Native `TextInput` prop on it.

# Navigation in React Native

## Introduction using `useRouter`:
### The Basics of Routing

Routing is how your app handles navigating from one screen to another. In mobile development, routing relies on a **stack architecture** (think of a stack of physical cards).

- **Push:** When you navigate to a new screen, you "push" a new card onto the top of the stack.
    
- **Pop (Back):** When you press the back button, you "pop" the top card off, revealing the screen underneath.
    
- **Deep Linking:** Routing also handles mapping standard URLs (like `myapp://profile/123`) to specific screens in your app.
    

---

### Context: `useRouter` vs `useNavigation`

To be absolutely clear: the `useRouter` hook is specific to **Expo Router** (the file-based routing system for React Native, similar to Next.js). If you are using vanilla React Navigation without Expo Router, the equivalent hook is `useNavigation`.

Assuming you are using Expo Router, here is how `useRouter` works.

---

### `useRouter` API & Common Methods

First, import and initialize the hook inside your component:

JavaScript

```
import { useRouter } from 'expo-router';

export default function MyScreen() {
  const router = useRouter();
  // ...
}
```

Here are the essential methods you need to know:

- **`router.push(href)`**
    
    - **What it does:** Pushes a new route onto the navigation stack.
        
    - **When to use it:** Standard navigation where you want the user to be able to swipe back or press the back button to return to the current screen.
        
    - **Example:** `router.push('/profile/123')`
        
- **`router.replace(href)`**
    
    - **What it does:** Replaces the current route in the navigation stack with a new one.
        
    - **When to use it:** When you _don't_ want the user to go back to the previous screen. The classic example is replacing a "Login" screen with the "Home" screen after a successful login.
        
    - **Example:** `router.replace('/home')`
        
- **`router.back()`**
    
    - **What it does:** Pops the current screen off the stack, navigating to the previous screen.
        
    - **When to use it:** Custom back buttons or closing modals. If there is no history to go back to, it does nothing.
        
    - **Example:** `router.back()`
        
- **`router.navigate(href)`**
    
    - **What it does:** Routes to the specified screen. If the screen is already in the navigation hierarchy, it may jump to that existing screen rather than pushing a duplicate onto the stack (depending on your layout setup).
        
    - **When to use it:** General navigation, especially when dealing with tabs or drawers where you want to switch contexts rather than stack screens endlessly.
        
    - **Example:** `router.navigate('/settings')`
        
- **`router.setParams(params)`**
    
    - **What it does:** Updates the URL parameters for the currently active route without triggering a full screen change.
        
    - **When to use it:** Filtering lists, updating a search query, or changing a localized state that you want reflected in the URL/deep link.
        
    - **Example:** `router.setParams({ sort: 'descending' })`

## Navigation in React Native

React Native uses **React Navigation** library—there's no built-in navigation. There are **3 main navigation types**: Stack, Tab, and Drawer.

## Installation

```bash
npm install @react-navigation/native
npm install react-native-screens react-native-safe-area-context
```


## 1. [[Stack Navigation in ReactNative]]

Screens stack on top of each other like cards. Push new screens, pop to go back.[^2]

**Install:**

```bash
npm install @react-navigation/native-stack
```

**Setup:**

```javascript
import { NavigationContainer } from '@react-navigation/native';
import { createNativeStackNavigator } from '@react-navigation/native-stack';

const Stack = createNativeStackNavigator();

function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Home" component={HomeScreen} />
        <Stack.Screen name="Details" component={DetailsScreen} />
      </Stack.Navigator>
    </NavigationContainer>
  );
}
```

**Navigate:**

```javascript
navigation.navigate('Details')  // Go to screen
navigation.push('Details')      // Add duplicate screen
navigation.goBack()             // Go back one screen
navigation.pop()                // Same as goBack
navigation.popToTop()           // Go to first screen
```


## 2. [[Tab Navigation in ReactNative]]

Tabs at bottom of screen. Each tab is a different section/stack.

**Install:**

```bash
npm install @react-navigation/bottom-tabs
```

**Setup:**

```javascript
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';

const Tab = createBottomTabNavigator();

function App() {
  return (
    <NavigationContainer>
      <Tab.Navigator>
        <Tab.Screen name="Home" component={HomeScreen} />
        <Tab.Screen name="Settings" component={SettingsScreen} />
        <Tab.Screen name="Profile" component={ProfileScreen} />
      </Tab.Navigator>
    </NavigationContainer>
  );
}
```

Users tap tabs to switch between screens. Like Instagram or Twitter bottom navigation.[^4][^2]

## 3. Drawer Navigation

Slide-out menu from the side. Opens with hamburger icon or swipe.

**Install:**

```bash
npm install @react-navigation/drawer
npm install react-native-gesture-handler react-native-reanimated
```

**Setup:**

```javascript
import { createDrawerNavigator } from '@react-navigation/drawer';

const Drawer = createDrawerNavigator();

function App() {
  return (
    <NavigationContainer>
      <Drawer.Navigator>
        <Drawer.Screen name="Home" component={HomeScreen} />
        <Drawer.Screen name="Profile" component={ProfileScreen} />
        <Drawer.Screen name="Settings" component={SettingsScreen} />
      </Drawer.Navigator>
    </NavigationContainer>
  );
}
```

**Navigate:**

```javascript
navigation.openDrawer()   // Open drawer
navigation.closeDrawer()  // Close drawer
navigation.toggleDrawer() // Toggle open/close
```



## Combining Navigators

Real apps combine these. Common pattern: Drawer contains Tab navigator, each tab has Stack navigator.

**Example:**

```javascript
// Stack for Home tab
const HomeStack = createNativeStackNavigator();
function HomeStackScreen() {
  return (
    <HomeStack.Navigator>
      <HomeStack.Screen name="Home" component={HomeScreen} />
      <HomeStack.Screen name="Details" component={DetailsScreen} />
    </HomeStack.Navigator>
  );
}

// Stack for Settings tab
const SettingsStack = createNativeStackNavigator();
function SettingsStackScreen() {
  return (
    <SettingsStack.Navigator>
      <SettingsStack.Screen name="Settings" component={SettingsScreen} />
      <SettingsStack.Screen name="Profile" component={ProfileScreen} />
    </SettingsStack.Navigator>
  );
}

// Tabs containing stacks
const Tab = createBottomTabNavigator();
function TabNavigator() {
  return (
    <Tab.Navigator>
      <Tab.Screen name="HomeTab" component={HomeStackScreen} />
      <Tab.Screen name="SettingsTab" component={SettingsStackScreen} />
    </Tab.Navigator>
  );
}

// Drawer containing tabs
const Drawer = createDrawerNavigator();
function App() {
  return (
    <NavigationContainer>
      <Drawer.Navigator>
        <Drawer.Screen name="Tabs" component={TabNavigator} />
        <Drawer.Screen name="Help" component={HelpScreen} />
      </Drawer.Navigator>
    </NavigationContainer>
  );
}
```

This gives you: drawer menu, bottom tabs, and each tab has multiple screens.[^1][^6]

## ![[New Navigation System in Expo React Native]]

# Button in React Native

React Native has a built-in `Button` component for basic clickable buttons. It's simple but has limited styling options, so for custom designs you'll use `Pressable` or `TouchableOpacity` instead.[^1][^2]

## Built-in Button

Import from React Native:[^3]

```javascript
import { Button } from 'react-native';
```

Basic usage:[^4][^2]

```javascript
<Button 
  onPress={handlePress}
  title="Click Me"
  color="#841584"
/>
```


### Required Props

- **onPress** - Function that runs when button is pressed. Gets called with a press event object[^2]
- **title** - Text displayed on the button. On Android it's automatically uppercased[^2]


### Optional Props

- **color** - Text color on iOS, background color on Android. Default is blue (\#2196F3 Android, \#007AFF iOS)[^2]
- **disabled** - Boolean to disable button interactions. Default is `false`[^2]
- **accessibilityLabel** - Text for screen readers


## Why Button Sucks

The built-in `Button` has almost zero customization. You can't change padding, border radius, font size, or add icons. That's why everyone uses touchable components instead.[^5][^1][^4][^2]

# Touchable Components

### TouchableOpacity

Most common alternative. Reduces opacity when pressed to give visual feedback.[^6][^4]

```javascript
import { TouchableOpacity, Text } from 'react-native';

<TouchableOpacity 
  onPress={handlePress}
  activeOpacity={0.7}
  style={styles.button}
>
  <Text style={styles.text}>Press Me</Text>
</TouchableOpacity>
```

**activeOpacity** - Controls how transparent it gets when pressed. Value between 0-1, default is around 0.7.[^6]

### Pressable

Newer and more powerful component. Recommended by React Native team for new code.[^6]

```javascript
import { Pressable, Text } from 'react-native';

<Pressable 
  onPress={handlePress}
  style={({ pressed }) => [
    styles.button,
    pressed && styles.pressed
  ]}
>
  <Text>Press Me</Text>
</Pressable>
```

**Key difference**: `style` prop can be a function that receives `pressed` state. This lets you change styles dynamically when button is being pressed.[^6]

## When to Use What

- **Button** - Quick prototypes or very basic apps where styling doesn't matter[^4]
- **TouchableOpacity** - Simple custom buttons where opacity fade is enough feedback[^4][^6]
- **Pressable** - When you need custom press states, complex feedback, or future-proof code. Also supports Android ripple effect[^6]


## Common Pattern

```javascript
import { Pressable, Text, StyleSheet } from 'react-native';

function CustomButton({ onPress, title }) {
  return (
    <Pressable 
      style={styles.button}
      onPress={onPress}
    >
      <Text style={styles.text}>{title}</Text>
    </Pressable>
  );
}

const styles = StyleSheet.create({
  button: {
    backgroundColor: '#2196F3',
    padding: 12,
    borderRadius: 8,
    alignItems: 'center',
  },
  text: {
    color: 'white',
    fontSize: 16,
    fontWeight: 'bold',
  }
});
```

This wraps a `Text` component inside `Pressable` to create a fully customizable button.

# [[RAPI UI in ReactNative]]

## What is Rapi UI

Rapi UI is a React Native component library that provides pre-built UI components for building mobile apps. It's an open-source library created to speed up development by offering ready-to-use buttons, inputs, text components, and other common UI elements.[^1][^2][^3]

## Key Features

The library includes built-in dark mode support through a `ThemeProvider` component that wraps your app. This lets you toggle between light and dark themes without manually styling each component.[^5]

## Installation

Install the package and dependencies:[^5]

```bash
npm install react-native-rapi-ui
expo install react-native-safe-area-context expo-font @expo/vector-icons expo-asset
```

Wrap your app with the theme provider:[^5]

```javascript
import { ThemeProvider } from "react-native-rapi-ui";

const App = () => {
  return (
    <ThemeProvider theme="light">
      <Navigator />
    </ThemeProvider>
  );
};
```


## Current Statusmust

The library hasn't been updated in 5 years - last published version is 0.2.1. This means it's likely outdated and may not work well with current React Native versions. Consider using more actively maintained UI libraries like React Native Paper, NativeBase, or React Native Elements instead for production projects.

# Ionicons in React Native

Ionicons are icon sets you can use in React Native through the `react-native-vector-icons` library. They're scalable vector icons that look sharp on any screen size.[^9][^10][^11]

## installation

## Basic Usage

```javascript
import { Ionicons } from "@expo/vector-icons";

<Ionicons name="home-outline" size={size} color={color} />
```


## Common Icon Names

**Navigation \& UI:**

- `home` - Home icon
- `menu` - Hamburger menu
- `search` - Search/magnifying glass
- `settings` - Gear icon
- `close` - X close button
- `arrow-back` - Back arrow
- `arrow-forward` - Forward arrow
- `chevron-down` - Down chevron
- `chevron-up` - Up chevron
- `add` - Plus sign
- `remove` - Minus sign
- `trash` - Delete/trash icon
- `create` - Edit/pencil icon

**Social \& Communication:**

- `heart` - Heart icon (favorite)
- `heart-outline` - Empty heart
- `star` - Filled star
- `star-outline` - Empty star
- `share` - Share icon
- `send` - Send/paper plane
- `call` - Phone icon
- `mail` - Email/envelope
- `chatbubble` - Chat/message
- `notifications` - Bell icon
- `person` - User profile
- `people` - Multiple users

**Media \& Content:**

- `camera` - Camera icon
- `image` - Picture/gallery
- `play` - Play button
- `pause` - Pause button
- `refresh` - Refresh/reload
- `download` - Download arrow
- `cloud-upload` - Upload icon
- `attach` - Attachment/paperclip
- `document` - File/document
- `folder` - Folder icon
- `calendar` - Calendar icon
- `time` - Clock icon

**Actions:**

- `checkmark` - Checkmark/tick
- `checkmark-circle` - Circle with check
- `close-circle` - Circle with X
- `information-circle` - Info icon
- `warning` - Warning/alert
- `eye` - View/visible
- `eye-off` - Hidden/invisible
- `lock-closed` - Lock/secure
- `lock-open` - Unlocked

You can browse all icons at https://ionic.io/ionicons

## Common Props

**`name`** (required) - The icon name:[^6][^7]

```javascript
<Icon name="home" />
```

**`size`** - Icon size in pixels (default: 30):[^7][^6]

```javascript
<Icon name="home" size={50} />
```

**`color`** - Icon color (any valid color string):[^6][^7]

```javascript
<Icon name="home" color="blue" />
<Icon name="home" color="#FF5733" />
```

**`style`** - Apply additional styles:[^9]

```javascript
<Icon name="home" style={{ marginTop: 10 }} />
```


## Platform-Specific Icons

Use different icons for iOS vs Android:[^7][^6]

```javascript
<Icon 
  ios="ios-home" 
  android="md-home" 
  size={30} 
  color="blue" 
/>
```

This shows `ios-home` on iOS and `md-home` on Android.[^6]

## Complete Example

```javascript
import React from 'react';
import { View, Text, TouchableOpacity } from 'react-native';
import Icon from 'react-native-vector-icons/Ionicons';

const App = () => {
  return (
    <View style={{ padding: 20 }}>
      <Text>Icon Examples:</Text>
      
      {/* Basic icon */}
      <Icon name="home" size={40} color="blue" />
      
      {/* Icon in button */}
      <TouchableOpacity style={{ flexDirection: 'row', alignItems: 'center' }}>
        <Icon name="search" size={20} color="white" />
        <Text>Search</Text>
      </TouchableOpacity>
      
      {/* Multiple icons */}
      <View style={{ flexDirection: 'row' }}>
        <Icon name="heart-outline" size={30} color="red" />
        <Icon name="star" size={30} color="gold" />
        <Icon name="share" size={30} color="green" />
      </View>
      
      {/* Inline icon with text */}
      <Text>
        Click here <Icon name="arrow-forward" size={16} /> to continue
      </Text>
    </View>
  );
};

export default App;
```


## Most Useful Parameters Summary

| Prop | Type | Default | Description |
| :-- | :-- | :-- | :-- |
| `name` | string | required | Icon name [^6] |
| `size` | number | 30 | Icon size [^6] |
| `color` | string | black | Icon color [^6] |
| `style` | object | - | Additional styles [^9] |
| `ios` | string | - | iOS-specific icon [^7] |
| `android` | string | - | Android-specific icon [^7] |

Ionicons are perfect for navigation bars, buttons, and anywhere you need simple, clean icons that scale properly.


# Hooks

## useState

**useState** is a React hook that lets you add state variables to functional components. State is data that changes over time and causes your component to re-render when updated.

### Syntax

```javascript
const [state, setState] = useState(initialValue);
```

- `state`: Current value of your state variable[^5]
- `setState`: Function to update the state and trigger re-render[^5]
- `initialValue`: Starting value for the state[^1]


### Example

```javascript
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

When you click the button, `setCount` updates the state and React re-renders the component with the new value.[^6]

## useEffect

**useEffect** is a hook for handling side effects in functional components. Side effects include data fetching, subscriptions, timers, or manually changing the DOM.

### Syntax

```javascript
useEffect(() => {
  // Your side effect code here
  return () => {
    // Cleanup (optional)
  };
}, [dependencies]);
```

The function runs after every render by default, i.e, 
The dependency array controls when it runs: 
	empty array `[]` means it runs once on mount, specific values mean it runs when those values change.

| **Scenario**          | **Syntax**                                | **When it runs**                                       |
| --------------------- | ----------------------------------------- | ------------------------------------------------------ |
| **No Array**          | `useEffect(() => { ... })`                | After **every** render and re-render.                  |
| **Empty Array**       | `useEffect(() => { ... }, [])`            | Only **once**, after the initial mount.                |
| **With Dependencies** | `useEffect(() => { ... }, [prop, state])` | On mount, and **only when** `prop` or `state` changes. |

### Example

```javascript
import { useState, useEffect } from 'react';

function DataFetcher() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('https://api.example.com/data')
      .then(response => response.json())
      .then(result => setData(result));
  }, []); // Runs once on mount

  return <div>{data ? data : 'Loading...'}</div>;
}
```

This fetches data once when the component mounts.[^10][^6]

## createContext

**createContext** creates a Context object that lets you pass data through the component tree without manually passing props at every level. You use it with `useContext` hook to consume the context values.

### Syntax

```javascript
const MyContext = createContext(defaultValue);
```


### Example

```javascript
import { createContext, useContext, useState } from 'react';

// Create context
const ThemeContext = createContext('light');

// Provider component
function App() {
  const [theme, setTheme] = useState('dark');

  return (
    <ThemeContext.Provider value={theme}>
      <ChildComponent />
    </ThemeContext.Provider>
  );
}

// Child component consuming context
function ChildComponent() {
  const theme = useContext(ThemeContext);
  return <div>Current theme: {theme}</div>;
}
```

### `<createContext().Provider>`
The `ThemeContext.Provider` wraps components that need access to the context, and `useContext(ThemeContext)` retrieves the current value without prop drilling.

`View` is the actual data that the passed as context.


# AppWrite with React Native

Appwrite is a complete backend-as-a-service (BaaS) that provides APIs for authentication, databases, storage, and serverless functions. When paired with React Native, it allows you to build cross-platform mobile apps rapidly without writing backend code.

## 1. High-Level Architecture

Before diving into code, it's important to understand how the pieces fit together. Your React Native app acts as the **Client**, making HTTP or WebSocket requests to the **Appwrite Server** (which can be hosted on Appwrite Cloud or self-hosted).

```
graph TD
    A[React Native App] -->|REST / GraphQL| B(Appwrite Client SDK)
    B -->|Auth API| C[User Sessions & OAuth]
    B -->|Database API| D[JSON Documents]
    B -->|Storage API| E[Images, Videos, Files]
    B -->|WebSocket| F[Realtime Subscriptions]
```

## 2. Installation & Prerequisites

To use Appwrite in a React Native (or Expo) environment, you need the official SDK and a URL polyfill. The polyfill is required because React Native's core Javascript engine lacks the standard browser `URL` API that Appwrite's internal networking relies on.

```
# Using npm
npm install react-native-appwrite react-native-url-polyfill

# Using Expo
npx expo install react-native-appwrite react-native-url-polyfill
```

## 3. The Central Configuration File

**Best Practice:** Never scatter Appwrite initialization across your app. Create a single `appwrite.js` (or `appwrite.ts`) file inside a `lib/` or `services/` folder. This ensures only **one instance** of the client is created.

**`lib/appwrite.js`**
```tsx
import 'react-native-url-polyfill/auto'; // MUST be at the very top!
import { Client, Account, Databases, Storage, ID, Query } from 'react-native-appwrite';

// 1. Define your configuration constants
export const appwriteConfig = {
    endpoint: '[https://cloud.appwrite.io/v1](https://cloud.appwrite.io/v1)',
    projectId: 'YOUR_PROJECT_ID',
    platform: 'com.yourcompany.yourapp', // e.g., your Android package name
    databaseId: 'YOUR_DATABASE_ID',
    userCollectionId: 'YOUR_USERS_COLLECTION_ID',
    storageId: 'YOUR_STORAGE_BUCKET_ID'
};

// 2. Initialize the Client
const client = new Client();

client
    .setEndpoint(appwriteConfig.endpoint)
    .setProject(appwriteConfig.projectId)
    .setPlatform(appwriteConfig.platform);

// 3. Initialize the Services
export const account = new Account(client);
export const databases = new Databases(client);
export const storage = new Storage(client);

// Export utilities for easy access
export { ID, Query, client };
```

## 4. Authentication (Accounts)

Appwrite handles secure password hashing, session management, and cookies/tokens automatically.

### Sign Up (Register)

To create a new user, you use `account.create()`. You must provide a unique ID (Appwrite provides a helper for this), email, and password.

```tsx
import { account, ID } from '../lib/appwrite';

const registerUser = async (email, password, name) => {
    try {
        // ID.unique() generates a random 20-character string
        const newAccount = await account.create(ID.unique(), email, password, name);
        console.log('User registered!', newAccount);
        
        // Best practice: Log them in immediately after creating the account
        await loginUser(email, password); 
    } catch (error) {
        console.error('Registration Failed:', error.message);
    }
};
```

### Sign In (Login) & Sign Out

Logging in creates a secure session. In React Native, this session token is securely stored on the device automatically by the SDK.

```tsx
// Login
const loginUser = async (email, password) => {
    try {
        const session = await account.createEmailPasswordSession(email, password);
        console.log('Session created:', session);
    } catch (error) {
        console.error('Login Failed:', error.message);
    }
};

// Get Current Logged-In User
const getCurrentUser = async () => {
    try {
        const currentAccount = await account.get();
        return currentAccount;
    } catch (error) {
        // If no session exists, this will throw an error
        return null; 
    }
};

// Logout
const logoutUser = async () => {
    try {
        // 'current' deletes the session for the device currently making the request
        await account.deleteSession('current'); 
    } catch (error) {
        console.error('Logout Failed:', error);
    }
};
```

## 5. Databases (CRUD Operations)

Appwrite's Database structure is: **Database -> Collection -> Document**.

Collections are like tables, and Documents are like rows (stored as JSON).

### Creating a Document

When you save data, you point it to a specific Database and Collection.

```tsx
import { databases, appwriteConfig, ID } from '../lib/appwrite';

const createPost = async (title, content, authorId) => {
    try {
        const newPost = await databases.createDocument(
            appwriteConfig.databaseId,
            appwriteConfig.userCollectionId,
            ID.unique(), // Auto-generate document ID
            {
                title: title,
                content: content,
                author: authorId,
                createdAt: new Date().toISOString()
            }
        );
        return newPost;
    } catch (error) {
        console.error('Failed to create post:', error);
    }
};
```

### Querying Documents (Filtering, Sorting, Pagination)

Appwrite uses the `Query` class to filter results. You pass an array of Queries to the `listDocuments` method.

```tsx
import { databases, appwriteConfig, Query } from '../lib/appwrite';

const getRecentActivePosts = async () => {
    try {
        const posts = await databases.listDocuments(
            appwriteConfig.databaseId,
            appwriteConfig.userCollectionId,
            [
                Query.equal('status', 'active'),    // Filter: status == 'active'
                Query.orderDesc('createdAt'),       // Sort: newest first
                Query.limit(10),                    // Pagination: get only 10 items
                Query.offset(0)                     // Pagination: start from beginning
            ]
        );
        return posts.documents; // Returns an array of your data
    } catch (error) {
        console.error('Query failed:', error);
    }
};
```

## 6. Storage (File Uploads)

Uploading files from React Native requires a slightly specific format because mobile operating systems handle files differently than web browsers.

### Uploading an Image

If you use a library like `expo-image-picker`, you get a URI. You must format this into an object Appwrite can understand.

```tsx
import { storage, appwriteConfig, ID } from '../lib/appwrite';

// Assuming 'file' comes from expo-image-picker
const uploadImage = async (file) => {
    if (!file) return;

    // React Native specific file formatting
    const asset = {
        name: file.fileName || 'image.jpg',
        type: file.mimeType || 'image/jpeg',
        size: file.fileSize,
        uri: file.uri,
    };

    try {
        const uploadedFile = await storage.createFile(
            appwriteConfig.storageId,
            ID.unique(),
            asset
        );
        
        // Get the viewable URL for the image
        const fileUrl = storage.getFilePreview(
            appwriteConfig.storageId, 
            uploadedFile.$id,
            2000, // width
            2000, // height
            'top', // gravity
            100 // quality
        );

        return fileUrl;
    } catch (error) {
        console.error("Upload failed", error);
    }
};
```

## 7. Realtime (WebSockets)

Realtime allows you to update your React Native UI instantly when data changes in your database without the user having to "pull to refresh".

You use `client.subscribe()` and listen to a "Channel". Channels are specific strings that point to resources (like a whole collection, or just one document).

```
import { client, appwriteConfig } from '../lib/appwrite';
import { useEffect } from 'react';

// Inside a React Native Component
useEffect(() => {
    // Construct the channel string for a specific collection
    const channel = `databases.${appwriteConfig.databaseId}.collections.${appwriteConfig.userCollectionId}.documents`;

    // Subscribe to the channel
    const unsubscribe = client.subscribe(channel, (response) => {
        
        // response.events contains what happened (create, update, delete)
        if (response.events.includes('databases.*.collections.*.documents.*.create')) {
            console.log('A new document was just added!', response.payload);
            // Update your React State here to show the new data instantly
        }
        
        if (response.events.includes('databases.*.collections.*.documents.*.delete')) {
            console.log('A document was deleted!', response.payload);
        }
    });

    // Cleanup subscription when the component unmounts!
    return () => {
        unsubscribe();
    };
}, []);
```

## 8. State Management Architecture

In a real React Native application, you shouldn't call Appwrite directly from your UI buttons. Instead, wrap Appwrite logic in a **React Context** (or Zustand/Redux).

This allows you to access the `user` and `isLoading` states from anywhere in your app to protect screens (e.g., redirecting to Login if `user === null`).

```
sequenceDiagram
    participant UI as React Native UI
    participant Context as Global Context API
    participant SDK as Appwrite SDK (lib/appwrite.js)
    
    UI->>Context: Call login(email, password)
    Context->>SDK: account.createEmailPasswordSession()
    SDK-->>Context: Returns Session
    Context->>SDK: account.get() (Fetch User Data)
    SDK-->>Context: Returns User Profile
    Context-->>UI: State Updates -> UI Re-renders
```

### Summary Checklist for a Production App:

1. [ ] Check if a user is logged in on app startup (Splash Screen).
    
2. [ ] Use Appwrite permissions to ensure users can only edit their own documents.
    
3. [ ] Handle Appwrite exceptions gracefully (e.g., showing a Toast message for `error.message` if a login fails).
    
4. [ ] Unsubscribe from Realtime events when screens are closed to save battery and memory.