AMD Packager PHP
================

A Tool to optimize your [AMD JavaScript Modules](https://github.com/amdjs/amdjs-api/wiki/AMD).
It will find the dependencies and package it into one file.

CLI
---

	packager-cli.php [options] <modules>

	Options:
	  -h --help             Show this help
	  --options             Specify another options file (defaults to options.php)
	  -o --output           The file the output should be written to
	  --modules --list      List the modules
	  --dependencies        List the dependencies map
	  --graph               Create a structural dependency graph
							and write it to this file
	  --watch               Watches the required modules


### Tip:

Add the following line to your `~/.bashrc` file

	alias packager-amd='~/path/to/amd-packager-php/packager-cli.php'

Now you can use the packager as:

	packager-amd Package/Module1 Package/Module2 > myTools.js


Packager Class
--------------

### API Docs

[API documentation](http://arian.github.com/amd-packager-php/) generated by phpdoc

### Example

``` php

$packager = new Packager;

// Set baseurl

$packager->setBaseUrl(__DIR__ . '/../foo/bar');

// Add aliases

$packager->addAlias('Core', 'MooTools/Core');
$packager->addAlias('Tests', 'MooTools/Tests');

// Require. Returns a Packager_Builder instance

$builder = $packager->req(array('Core/DOM/Node', 'Tests/Host/Array'));

// Output

echo '/* These modules are loaded:';
echo implode("\n - ", $builder->modules());
echo "\n*/\n\n";

echo $builder->output();

```

Unit Tests
----------

See the `test` folder.
Run it with `phpunit test` for the PHP Packager.
For the resulted JavaScript and the Loader.js you could open
`test/js/SpecRunner.html` in your browser or run `test/js/SpecRunner.js` with
Node.js.

Web Interface
-------------

The Web Interface of [Packager Web](https://github.com/kamicane/packager-web)
has been ported to work with this packager. This means you can easily select
the desired modules and build them in one or several files. The download also
supports compressing of the JavaScript with YUI-compressor or UglifyJS. When
the files are packaged per package, the download is provided as a ZIP file.

JavaScript Examples:
--------------------

This are examples of your source files. You can already define an ID for the
module, but that's not very useful. The dependencies argument can be relative
paths to the other modules, or use the aliases.

**Source/Storage.js**: Only the factory function

``` javascript
define(function(){
	var storage = {};
	return {
		store: function(key, value){
			storage[key] = value;
			return this;
		},
		retrieve: function(key){
			return storage[key];
		}
	};
});
```

**Source/App.js**: With dependencies

``` javascript
define(['Core/Utility/typeOf', './Storage.js'], function(typeOf, Storage){
	Storage.store('foo', 'bar');
	alert(storage.retrieve('foo')); // bar
});
```

After that you can write a build script or use the CLI script.
The packager will add an ID to each `define()` function an ID so when each
`define()` is in the same file, everything continues to work. If the module
already had an ID, it will not replace it.


Using `has()`
-------------

[has.js](https://github.com/phiggins42/has.js) is a great tool for feature detections
and has a nice API which makes it easy to strip unnecessary code and which works
on the client-side as well.

**Example of using has()**

``` javascript
define(function(){

	if (has('css-transitions')){
		// do something cool
	} else {
		// meh...
	}

});
```

Now if you *know* your script will only be used in modern browsers, the `else`
code-branch is useless.

``` php
$packager = new Packager;
$builder = $packager->req(array('ModuleA', 'ModuleB'));

$builder->addHas('css-transitions', true);
$builder->output();
```

This will result in code like:

``` javascript
if (true){
	// do something cool
} else {
	// meh...
}
```

Minifiers, like UglifyJS or Google Closure Compiler, can remove dead-code branches
like these. This will make your build perfectly optimized for your environment.

Analysis
--------

With the `--graph` cli option, a dependency graph can be made. These can be
generated to do some analysis on your modules and make better decisions.

Alternatively the `Packager_Graph` class in Graph.php can be used directly too.
For this graph [Graphviz](http://www.graphviz.org/) is required.

![Dependency Graph](https://raw.github.com/arian/amd-packager-php/master/test/php/hello.png)


Notes
-----

This is not a full implementation of the AMD specification.

Some restrictions are:

- It does not execute JavaScript, so the `define` function MUST be in the literal form. It also MUST use square brackets (`[` and `]`) for dependencies.


Requirements
------------

- PHP 5.2 (tested on 5.3, but _should_ work on 5.2)
- Graphviz (for the `--graph` option or Graph.php class)

To installation for graphviz is easy for debian(-like) systems with:

	sudo apt-get install graphviz

For other systems you should look at the [graphviz download page](http://www.graphviz.org/Download..php).

License
-------

Just the MIT-License
