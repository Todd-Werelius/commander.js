# cmdline.js

  A solution for [node.js](http://nodejs.org) simple command-line parameter parsing and full command-line program support forked from [commander](https://github.com/visionmedia/commander). 
  
  This fork provides emphasis on providing more straight forward ( but slightly less flexible ) command line parameter handling and optional user shutdown handling.
  

 [![Build Status](https://api.travis-ci.org/visionmedia/commander.js.svg)](http://travis-ci.org/visionmedia/commander.js)

## Installation

    $ npm install cmdline

## Parsing command line arguments example
```js
#!/usr/bin/env node

var cfg = require('cmdline');

cfg
  .version('0.0.1')
  .usage(' < --url > [-S] [-P] [-T]','node server')
  .param('-U, --url <URL>', 'URL to parse')                             // A required param 
  .param('-S, --stripjs', 'Strips out <scripts> from html')              // A switch 
  .param('-P, --port [PORT]', 'Port server listens on, default is 3000') // Optional param with default
  .param('-T, --timeout [0]' , 'ms to wait, default is no limit')        // A optional param with no deefault value 
.parse(process.argv, [function(error)]{
    // if error is set then soemthing went wrong 
    // else configure to run ... 
});

```

## Method Explantion  

The method explanations assume that the program file is server.js 

### `.version('0.0.2')`

Sets the version. When the -V or --version parameter switch is set this will output to the console and the program will exit

```
$ node server -V
v0.0.2
```

### `.usage(desc,[name])` 
Sets usage instructions. 'desc' tells the user one or more ways to use the program, optionally you can use 'name'  to overide the program name pre-fix.  When -H or --help switches are used this will be the first line outputed in the automated help

```
examples;

.usage('-u URL [-p port]')
  $ node server -h
  
    Usage: server -u URL [-p port]
    .
    .
    .

.usage('-u URL [-p port]','node server.js')
  $ node server -h
  
    Usage: node server.js -u URL [-p port]
    .
    .
    .
  
```

### `.param(flag,desc,[fn],[defValue])`

Creates a named parameter. The value of the parmeter will be atteched to the cmdline object using the long name of the parameter and can be read using . notation 

ex: cfg.url, cfg.stripjs , cfg.port, cfg.timeout etc.  


#### flag 
A string in the format of `'-shortname, --longname [optional rule]'` the rule identifies the parameter as either a switch, a required value or an optional value. The content within a rule is passed to [fn] if defined but has no meaning to the cmdline parser and is typicaly just used to describe the expected value. 

`parameter('-P, -port <number>',...)` trailing <...> indicates required value

`parameter('-T, -timeout [miliseconds]',...)`  trailing [...] indicates optional value

`parameter('-S, --stripjs',...)` empty ( no <> or [] ) indicates a switch

A required parameter MUST be set by the caller or an error will be generated
  [defValue] is ignored
  [fn] is used if defined  
  
A optional parameter is not required
  [defValue] or [fn] if defined will be used to set the value
  [defValue] or [fn] if UNdefined value will be set to null

A switch parameter if present has it's value set to true, if missing the parmeter will be set to false
  [defValue] is ignored
  [fn] is executed if defined but the return value is not used  

 Short flags may be passed as a single arg, for example `-abc` is equivalent to `-a -b -c`. Multi-word options such as "--template-engine" are camel-cased, becoming `templateEngine` etc.



## Automated --help

 The help information is auto-generated based on the information commander already knows about your program, so the following `--help` info is for free:

```  
 $ ./examples/pizza --help

   Usage: pizza [options]

   Options:

     -V, --version        output the version number
     -p, --peppers        Add peppers
     -P, --pineapple      Add pineapple
     -b, --bbq            Add bbq sauce
     -c, --cheese <type>  Add the specified type of cheese [marble]
     -h, --help           output usage information

```

## Coercion

```js
function range(val) {
  return val.split('..').map(Number);
}

function list(val) {
  return val.split(',');
}

function collect(val, memo) {
  memo.push(val);
  return memo;
}

function increaseVerbosity(v, total) {
  return total + 1;
}

program
  .version('0.0.1')
  .usage('[options] <file ...>')
  .option('-i, --integer <n>', 'An integer argument', parseInt)
  .option('-f, --float <n>', 'A float argument', parseFloat)
  .option('-r, --range <a>..<b>', 'A range', range)
  .option('-l, --list <items>', 'A list', list)
  .option('-o, --optional [value]', 'An optional value')
  .option('-c, --collect [value]', 'A repeatable value', collect, [])
  .option('-v, --verbose', 'A value that can be increased', increaseVerbosity, 0)
  .parse(process.argv);

console.log(' int: %j', program.integer);
console.log(' float: %j', program.float);
console.log(' optional: %j', program.optional);
program.range = program.range || [];
console.log(' range: %j..%j', program.range[0], program.range[1]);
console.log(' list: %j', program.list);
console.log(' collect: %j', program.collect);
console.log(' verbosity: %j', program.verbose);
console.log(' args: %j', program.args);
```

## Custom help

 You can display custom `-h, --help` information
 by listening for "--help". Commander will automatically
 exit once you are done so that the remainder of your program
 does not execute causing undesired behaviours, for example
 in the following executable "stuff" will not output when
 `--help` is used.

```js
#!/usr/bin/env node

/**
 * Module dependencies.
 */

var program = require('../');

function list(val) {
  return val.split(',').map(Number);
}

program
  .version('0.0.1')
  .option('-f, --foo', 'enable some foo')
  .option('-b, --bar', 'enable some bar')
  .option('-B, --baz', 'enable some baz');

// must be before .parse() since
// node's emit() is immediate

program.on('--help', function(){
  console.log('  Examples:');
  console.log('');
  console.log('    $ custom-help --help');
  console.log('    $ custom-help -h');
  console.log('');
});

program.parse(process.argv);

console.log('stuff');
```

yielding the following help output:

```

Usage: custom-help [options]

Options:

  -h, --help     output usage information
  -V, --version  output the version number
  -f, --foo      enable some foo
  -b, --bar      enable some bar
  -B, --baz      enable some baz

Examples:

  $ custom-help --help
  $ custom-help -h

```

## .outputHelp()

  Output help information without exiting.

## .help()

  Output help information and exit immediately.

## Links

 - [API documentation](http://visionmedia.github.com/commander.js/)
 - [ascii tables](https://github.com/LearnBoost/cli-table)
 - [progress bars](https://github.com/visionmedia/node-progress)
 - [more progress bars](https://github.com/substack/node-multimeter)
 - [examples](https://github.com/visionmedia/commander.js/tree/master/examples)

## License 

(The MIT License)

Copyright (c) 2011 TJ Holowaychuk &lt;tj@vision-media.ca&gt;

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
'Software'), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be
included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
