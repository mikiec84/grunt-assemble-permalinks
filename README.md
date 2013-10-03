# permalinks [![NPM version](https://badge.fury.io/js/permalinks.png)](http://badge.fury.io/js/permalinks) 

> Permalinks plugin for Assemble, enables powerful and configurable URI patterns, uses [Moment.js](http://momentjs.com/) for parsing dates, and lots more.


## Quickstart

From the same directory as your project's [Gruntfile][Getting Started] and [package.json][], install this plugin with the following command:

```bash
npm install permalinks --save-dev
```

Once that's done, just add `permalinks`, the name of this module, to the `plugins` option in the Assemble task:

```js
module.exports = function(grunt) {

  // Project configuration.
  grunt.initConfig({
    assemble: {
      options: {
        plugins: ['permalinks'],
        permalinks: {
          pattern: ':year/:month/:day/foo:/index.html'
        }
      },
      ...
    }
  });
  grunt.loadNpmTasks('assemble');
  grunt.registerTask('default', ['assemble']);
};
```

If everything was installed and configured correctly, you should be ready to go!




## Options
### dateFormats
Type: `Array`
Default: `["YYYY-MM-DD"]`

Array of custom date formats for [Moment.js](http://momentjs.com/) to use for parsing dates.


```js
options: {
  permalinks: {
    dateFormats: ["YYYY-MM-DD", "MM-DD-YYYY", "YYYY-MM-DDTHH:mm:ss.SSS"]
  }
  ...
}
...
```


### lang
Type: `String`
Default: `en`

Set the "global" language for [Moment.js](http://momentjs.com/) to use for converting dates:

```js
options: {
  permalinks: {
    pattern: ':year/:month/:day/:basename:ext',
    lang: 'fr'
  }
  files: {
    'blog/': ['templates/blog/*.hbs']
  }
}
...
//=> blog/2013/mars/13/my-post.html
```

### exclusions
Type: `Array`
Default: `['_page', 'data', 'filePair', 'page', 'pageName']`

Properties to omit from the context for processing replacement patterns. I wanted to use this for omitting the default properties, but I decided to expose this as an option in case it comes in useful to someone else.

```js
options: {
  permalinks: {
    exclusions: ["foo", "bar"],
  }
  ...
}
...
```



## Patterns
### Permalink Patterns
> Replacement patterns for dynamically constructing permalinks, and thus directory structures.

Permalinks are **appended to the dest directory**. So given this config:

```js
assemble: {
  blog: {
    options: {
      permalinks: {
        pattern: ':year/:month/:day/:basename:ext'
      }
    },
    files: {
      'blog/archives/': ['archives/*.hbs']
    }
  }
}

// the generated directory structure and resulting path would look something like:
//=> 'blog/archives/2011/01/01/an-inspiring-post.html'
```

### How patterns work

This plugin comes with a number of built-in replacement patterns that will automatically parse and convert the patterns into the appropriate string. Assemble provides a number of generic variables for accessing data from page, such as `basename`, `ext`, `filename` and so on. This plugin simply dynamically builds the replacement patterns from those generic variables, so barring a few exceptions (`_page`, `data`, `filePair`, `page`, `pageName`), you should be able to use any _applicable_ variable that is on the page context in your replacement patterns.

Such as:

* `:ext`: The extension of the file, for example `.html`
* `:extname`: Alias for `:ext`.
* `:basename`: A slugified version of the title of the file. So `My Very first Post` becomes `my-very-first-post`.
* `:filename`: A slugified version of the name of the dest file. So `My Very first Post` becomes `my-very-first-post.html`.
* `:pagename`: Alias for `:filename`.
* `:category`: A slugified version of the very first category for a page.


#### Custom Patterns

If you have some patterns you'd like to implement, if you think they're common enough that they should be built into this plugin, please submit a pull request.

Adding patterns is easy, just add a `replacements: []` property to the `permalinks` option, then add any number of patterns to the array. For example, let's say we want to add the `:author` variable to our permalinks:

```js
options: {
  permalinks: {
    pattern: ':year/:month/:day/:author/:slug:ext',
    replacements: []
  }
}
...
```

Since `:authors` is not a built-in variable, we need to add a replacement pattern so that any permalinks that include this variable will actually work:

```js
options: {
  permalinks: {
    pattern: ':year/:month/:day/:author/:slug:ext',
    replacements: [
      {
        pattern: ':author',
        replacement: '<%= pkg.author.name %>'
      }
    ]
  }
}
...
```
#### with custom properties

Any string pattern is acceptable, as long as a `:` precedes the variable, but don't forget that there must also be a matching property in the context or Assemble will might an error (or worse, not). In other words, when you add a replacement pattern for `:foo`, it's good practice to make sure this property exists:

```yaml
---
foo: bar
slug:
---
```


### [Moment.js](http://momentjs.com/) date patterns

> This plugin uses the incredibly feature rich and flexible [moment.js](http://momentjs.com/) for parsing dates. If you have a feature request, please don't hesitate to create an issue or make a pull request.

#### Common date patterns

* `:year`: The year of the date, four digits, for example `2014`
* `:month`: Month of the year, for example `01`
* `:day`: Day of the month, for example `13`
* `:hour`: Hour of the day, for example `24`
* `:minute`: Minute of the hour, for example `01`
* `:second`: Second of the minute, for example `59`

For the following examples, let's assume we have a date in the YAML front matter of a page formatted like this:

```yaml
---
date: 2014-01-29 3:45 PM
---
```
(_note that this property doesn't have to be in YAML front matter, it just needs to be in the `page.data` object, so this works fine with `options.pages` collections as well._)

#### Full date
* `:date`:       Eqivelant to the full date: `YYYY-MM-DD`. Example: `2014-01-29`

#### Year
* `:YYYY`:      The full year of the date. Example: `2014`
* `:YY`:        The two-digit year of the date. Example: `14`
* `:year`:      alias for `YYYY`

#### Month name
* `:MMMM`:      The full name of the month. Example `January`.
* `:MMM`:       The name of the month. Example: `Jan`
* `:monthname`: alias for `MMMM`

#### Month number
* `:MM`:        The double-digit number of the month. Example: `01`
* `:M`:         The single-digit number of the month. Example: `1`
* `:month`:     alias for `MM`
* `:mo`:        alias for `M`

#### Day of the month
* `:day`:       alias for `DD`
* `:DD`:        The double-digit day of the month. Example: `29`
* `:D`:         The double-digit day of the month. Example: `29`

#### Day of the week
* `:dddd`:      Day of the week. Example: `monday`
* `:ddd`:       Day of the week. Example: `mon`
* `:dd`:        Day of the week. Example: `Mo`
* `:d`:         Day of the week. Example: `2`

#### Hour
* `:HH`:        The double-digit time of day on a 24 hour clock. Example `15`
* `:H`:         The single-digit time of day on a 24 hour clock. Example `3`
* `:hh`:        The double-digit time of day on a 12 hour clock. Example `03`
* `:h`:         The single-digit time of day on a 12 hour clock. Example `3`
* `:hour`:      alias for `HH`

#### Minute
* `:mm`:        Minutes. Example: `45`.
* `:m`:         Minutes. Example: `45`.
* `:min`:       Alias for `mm`|`m`.
* `:minute`:    Alias for `mm`|`m`.

#### Second
* `:ss`:        Seconds. Example: `09`.
* `:s`:         Seconds. Example: `9`.
* `:sec`:       Alias for `ss`|`s`.
* `:second`:    Alias for `ss`|`s`.



## Usage Examples
### Path separators

You don't have to use slashes (`/`) only in your permalinks, you can use `-` or `_` wherever you need them as well. For example, this is perfectly valid:

```
pattern: ':YYYY_:MM-:DD/:slug:category:foo/:bar/index.html'
```

**Warning**, this should be obvious, but make sure not to use a `.` in the middle of your paths, especially if you use Windows.


### Dynamically build slugs

You can even dynamically build up strings using Lo-Dash templates:

```yaml
---
date: 1-1-2014

## Dynamically build the slug for example
area: business
section: finance
slug: <%= area %>-<%= section %>
---
```
With this config:

```js
blog: {
  options: {
    permalinks: {
      pattern: ':year/:month/:day/:slug/:title.html'
    }
  },
  files: {
    'blog/': ['posts/*.hbs']
  }
}
```

Would render to:

```
blog/2014/01/01/business-finance/index.html
```

### More examples

```js
:year/:month/:day/:basename:ext
//=> dest + '/2014/01/01/my-post.html'

:year/:month/:day/:category/index.html
//=> dest + '/2014/01/01/javascript/index.html'

:year/:month/:day/:slug/index.html
//=> dest + '/2014/01/01/business-finance/index.html'
```




## Contributing
Please see the [Contributing to permalinks](https://github.com/helpers/permalinks/blob/master/CONTRIBUTING.md) guide for information on contributing to this project.

## Author

+ [github.com/jonschlinkert](https://github.com/jonschlinkert)
+ [twitter.com/jonschlinkert](http://twitter.com/jonschlinkert)

## License
Copyright (c) 2013 Jon Schlinkert, contributors.
Released under the MIT license

***

_This file was generated on Thu Oct 03 2013 16:33:08._
