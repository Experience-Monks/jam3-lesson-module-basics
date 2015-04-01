# Modules for Frontend JavaScript

This is a brief intro to modules for modern frontend development. It will introduce Node, npm, and browserify.

# <a id="sections" href="#sections">#</a> sections

- [concepts](#concepts)
  - [modules](#modules)
  - [dependencies](#dependencies)
  - [semantic versioning](#semantic-versioning)
- [tools](#tools)
  - [node](#node)
  - [npm](#npm)
  - [browserify](#browserify)
    - [basics](#basics)
    - [local modules](#local-modules)
- [further reading](#further-reading)

# <a id="concepts" href="#concepts">#</a> concepts

## modules

In this context, "modular" JavaScript refers to a piece of code that operates independently of the applications which surround it. A "module" — sometimes just a single function — is written, tested, and published in isolation, and often a good candidate for re-use across projects. 

The goal is to build *small programs that do one thing, do it well, and compose easily with other programs* (the Unix Philosophy). 

Some examples:

- [xhr](https://www.npmjs.com/package/xhr) - an XMLHttpRequest abstraction 
- [domready](https://www.npmjs.com/package/domready) - modern DOM ready wrapper
- [xtend](https://www.npmjs.com/package/xtend) - a function which merges objects

There are a few different formats for authoring and publishing modules, such as ES6, AMD, and CommonJS. This article will focus on CommonJS (used by Node). 

## dependencies

Each module can have a formalized set of "dependencies" — that is, other modules it needs to work. Dependencies can help reduce repetition and share code across many modules. The tooling will figure out how to download and consume these dependencies under the hood. In turn, those modules can have their own dependencies, and so forth.

For example, the earlier mentioned [xhr](https://www.npmjs.com/package/xhr) module has a dependency graph that looks a little like this, where each is a separate module: 

```ruby
xhr
├── once            # run a function once
└─┬ parse-headers   # parse http headers
  ├─┬ for-each      # forEach() but works on objects
  │ └── is-function # test if value is a function
  └── trim          # string trim utility
```

## semantic versioning

As you re-use the same piece of code across many projects, bugs and limitations with the original module begin to emerge. The code evolves and changes to better suit the new use cases, and sometimes it leads to a breaking API (i.e. renaming a public function). 

This is where semantic versioning can help us. A version is broken down into `major.minor.patch` numbers, separated by decimals: 

- `major` denotes a breaking change in the documented API (e.g. renaming or removing a public function)
- `minor` denotes a new feature was added in a backwards-compatible manner
- `patch` denotes a bug was fixed in a backwards-compatible manner

Modules typically start at version `1.0.0`. When you bump one number, you must reset the numbers to the right of it back to zero. For example: 

- making a bug fix to `1.0.0` -> `1.0.1` (patch change)
- adding a new feature to `1.0.1` -> `1.1.0` (minor change)
- changing a method name in `1.1.0` -> `2.0.0` (major change)

When we install a module, we are also opting in for all of its `patch` and `minor` versions within the same `major` version. This way, we get the latest bug fixes, but our code won't break if a dependency decides to change its API. 

# <a id="tools" href="#tools">#</a> tools

To bring these concepts together, we will primarily be using these three open source tools:

- [node](https://nodejs.org/) - a platform built on Chrome's JavaScript engine (V8)
- [npm](https://npmjs.com/) - a package manager for JavaScript modules
- [browserify](https://github.com/substack/node-browserify) - a tool which bundles Node modules into something browsers understand

## node

The first step is to install Node; you can download it from [nodejs.org](https://nodejs.org/). Among other things, it provides a command-line interface for executing JavaScript source.

Once it's installed, open Terminal and try entering the following:

```sh
node -e "console.log(10 * 2)"
```

If all goes well, the above should print `20` (where `-e` is for eval).

![img](http://i.imgur.com/C7fOgy8.png)

<sup>(if that doesn't work, you may need to re-install Node [via homebrew](http://shapeshed.com/setting-up-nodejs-and-npm-on-mac-osx/))</sup>

Now you can run JavaScript files with `node path/to/file.js`, or start a REPL (read-eval-print-loop) with `node`. 

![repl](http://i.imgur.com/7KXYIBk.png)

<sup>**Tip:** You can quit the REPL with `Control + C`</sup>

This is a great way to quickly test some generic JavaScript code without booting up a browser. 

Since we are running in Node, we also have access to a standard library for things like File I/O and backend programming. 

For example, let's use Node's built-in [url](https://nodejs.org/api/url.html) module to parse the URL string  `"http://google.com/"`

![url](http://i.imgur.com/rQS9AeN.png)

Here we are *requiring* a built-in module called `url`. Require statements are part of Node, and look like this:

```js
var url = require('url');

//... use url.parse()
```

## npm

The next tool we need is [npm](http://npmjs.com/). If you installed Node through the site above, you should already have npm set up! We can test that with the following command:

```sh
npm -v
```

<sup>(if that doesn't work, you may need to re-install Node and npm [via homebrew](http://shapeshed.com/setting-up-nodejs-and-npm-on-mac-osx/))</sup>

Let's try installing our first module: [http-server](https://www.npmjs.com/package/http-server). This helps us get a static site up quickly without a bloated GUI tool like MAMP. 

```sh
npm install http-server -g
```

<sup>*Note:* The `-g` flag tells npm to install it globally.</sup>

If you get an `EACCESS` error, you need to fix your permissions:

[Fixing npm permissions](https://docs.npmjs.com/getting-started/fixing-npm-permissions#option-1-change-the-permission-to-npms-default-directory)  
[More Details (Stackoverflow)](http://stackoverflow.com/questions/16151018/npm-throws-error-without-sudo)  

Once it installs, you should be able to run it like so:

```sh
http-server -o
```

This should open the browser on `http://localhost:8080/`.

![http](http://i.imgur.com/1L9Dmcl.png)

Awesome! You just used your first module. This one is a command line tool, which is why we installed it globally with `-g`. When we install code dependencies (like [xhr](https://www.npmjs.com/package/xhr)), we typically install them locally.

## browserify

To bridge the gap between Node/npm and the browser, we will use a tool called [browserify](https://github.com/substack/node-browserify). It transforms Node `require()` statements into something the browser can execute, and bundles different modules into a single script. This allows us to use built-in Node modules like [url](https://nodejs.org/api/url.html), as well as modules on [npm](https://www.npmjs.com/). 

Close the earlier process (`Control + C`), and then stub out a folder where we can write some demos:

```sh
#go to your projects folder
cd ~/Documents

#make a new test folder
mkdir test-browserify

#move into that folder
cd test-browserify

#make an empty JavaScript file
touch index.js
```

<sup>*Note:* On Windows you might use `%HOMEPATH%/Documents` instead.</sup>

### basics

Open the empty `index.js` file we created above in your favourite editor (like [SublimeText](http://www.sublimetext.com/)). Add the following and save it:

```js
var url = require('url');

var parts = url.parse(window.location.href);
console.log(parts);
```

The above code won't work yet in the browser, since it uses Node APIs. For our demo, we will use [wzrd](https://github.com/substack/wzrd) to transform our Node `require()` statements into something the browser can understand. Install both tools like so:

```sh
npm install wzrd browserify -g
``` 

Now start `wzrd` on our source file, like so:

```sh
wzrd index.js
```

This will start a browserify development server on port `9966`. When `index.js` is requested, it will get "browserified" and transformed into something the browser can run. It will also serve you a basic `index.html` if you didn't write one, with a `<script>` tag for our `index.js`.

Now, when you open up `http://localhost:9966` in your browser and check the DevTools console, you'll see the Node code working as expected!

![console](http://i.imgur.com/wYaIGNK.png)

### local modules

The last thing we'll cover here is using a third-party module in our code. Let's install a SVG helper module, [svg-create-element](https://www.npmjs.com/package/svg-create-element).

```js
npm install svg-create-element
```

<sup>*Note:* We aren't installing this globally with `-g`. In later lessons we'll formalize this as a dependency with a `package.json`.</sup>

This will add the module into a folder called `node_modules` in our current directory. Now let's replace our `index.js` code with the following:

```js
//require the SVG helper module
var create = require('svg-create-element');

//create a SVG element
var svg = create('svg');

//add a polyline
var line = create('polyline', {
    stroke: 'orange',
    strokeWidth: 4,
    fill: 'transparent',
    points: [ 
      [ 50, 100 ], 
      [ 100, 50 ], 
      [ 150, 100 ], 
      [ 200, 50 ]
    ].join(' ')
});
svg.appendChild(line);

//add contnet to DOM
document.body.appendChild(svg);
```

And now, when we refresh our `http://localhost:9966/` page, we will see a sweet 2D line!

![svg](http://i.imgur.com/2wKR8ZJ.png)

# <a id="further-reading" href="#further-reading">#</a> further reading

This article only covers the basics of Node, npm, and getting modules working in the browser. A future lesson will go into more depth on the `require()` statement, and how we can build and publish our own modules. Until then, check out some of these links:

- [Module Best Practices](https://github.com/mattdesl/module-best-practices)
- [Browserify Help](http://browserify.org/articles.html)
- [npm docs](https://docs.npmjs.com/)
- [Node.js API docs](https://nodejs.org/api/)
- [Rapid Prototyping in JavaScript](http://mattdesl.svbtle.com/rapid-prototyping)