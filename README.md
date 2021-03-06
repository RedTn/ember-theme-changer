# ember-theme-changer

This Ember-Addon will help you to switch CSS style files on runtime.

For example, let's say you want to support multiple themes in your app and each of the theme styles is defined in its own CSS file (I'll show you how easy is to accomplish this on Ember). Ember-theme-changer addon provides an easy mechanism to change themes and subscribe to theme changes in your different components to handle those styles that you couldn't define using stylesheet.



## Installation

`ember install ember-theme-changer`


## Configuration

In our example we want to support Dark and Light themes. To accomplish it you must define Ember-Changer available themes in your `config/environment.js` file:

```
/* jshint node: true */
module.exports = function(environment) {
  var ENV = {
    /* ... */
    theme: {
      themes: [ 'light', 'dark'], // MANDATORY
      defaultTheme: 'dark', // OPTIONAL
      eventName: 'name of the event to be trigger when the theme changes', // OPTIONAL
      cookieName: 'name of cookie used to save the current theme value' // OPTIONAL
    }
  };
  return ENV;
};
```

## Defining themes on Ember.

There are many mechanism to define themes in your application. In the following example I'll be defining style variables but you can use any mechanism that works better for you. Also, I'll be using Sass preprocessor, but the same solution applies with Less:

### 1st) Writing multiple theme files

```
// app.scss

// In your app.scss file you can define styles that are dependent on the current theme, or import external styles too
@import "ember-modal-dialog/ember-modal-structure";
@import "ember-modal-dialog/ember-modal-appearance";

body {
  width: 100%
}
```

```
// main.scss

// here is where you define styles that depends on the current theme. All the theme dependent values are referenced with variables. eg:
body {
  background-color: $bodyBackgroundColor;
}
```

```
// dark.scss

// 1) define all the variables you need
$bodyBackgroundColor: '#fff';
...

// 2) Import your style fle where the variables will be consumed.
@import 'main';
```

And do the same for the rest of the themes. eg:
```
// light.scss

// 1) Define variables
$bodyBackgroundColor: '#000';
...
// 2) import style file
@import 'main';
```

### 2nd) Generating theme files.
By default, Ember only generates 1 css file based on your app.scss content (or app.less, app.css depending in your your style preprocessor), in this case we want to generate 2 extra files (dark.css & light.css). This is accomplish easily by telling Ember about the extra output files in the `ember-cli-build.js` file:

```
// ember-cli-build.js
    outputPaths: {
      app: {
        css: {
          'light': '/assets/light.css',
          'dark': '/assets/dark.css'
        }
      }
    },
```
As a result, ember will generate 3 css files (app.css, light.css & dark.css).

For more info, please read the official Ember-CLI doc: https://ember-cli.com/user-guide/#configuring-output-paths



## Usage

All interaction with the addon is though the `themeChanger` service.

### Change themes

You can circle through the themes invoking the service `toggleTheme` method, or set an specific value with the `theme` property.


```
// your_component.js:

themeChanger: service(),
....

action: {
  switchTheme() {
    this.get('themeSwitcher').toggleTheme();
  },
  setDarkTheme() {
    this.get('themeSwitcher').set('theme', 'dark');
  }
}
```

### Theme-change Event

In some situation you could have some styles defined outside of the CSS boundaries, this addon provides an event you can subscribe to detect changes int he current theme allowing you to execute any code.


```
// your_component.js:
themeChanger: service(),
chartColor,
....

  didReceiveAttrs() {
    this._super(...arguments);

    // subscribe to the theme changes event
    this.get('themeChanger').onThemeChanged((theme) => {

      // your code to handle theme changes:
      if (theme === 'light') {
        this.set('chartColor', 'black');
      } else {
        this.set('chartColor', 'white');
      }
      ...
    });

  },


  // Its important to unsubscribe to the event when your component will be destroyed
  willDestroyElement() {
    this.get('themeChanger').offThemeChanged();
    this._super(...arguments);
  }
```