# Commander.js

  The complete solution for [node.js](http://nodejs.org) command-line interfaces, inspired by Ruby's [commander](https://github.com/visionmedia/commander).

 [![Build Status](https://api.travis-ci.org/visionmedia/commander.js.svg)](http://travis-ci.org/visionmedia/commander.js)

## Installation

    $ npm install commander

## Option parsing

 Options with commander are defined with the `.option()` method, also serving as documentation for the options. The example below parses args and options from `process.argv`, leaving remaining args as the `program.args` array which were not consumed by options.
 
The option method's signature is  

```js
  .option( flags,  // A string describing the options short and long names as well as an optional modifer 
                   // that assigns rules to the option  
                   //
                   // <type> required -- option must be present on command line, [def] if provided will be ignored 
                   // [type] optional -- option may be excluded, [def] should be provided else the value will be
                   // undefined
                   //
                   // 'type' has no effect on how the option is processed but is passed to [fn] if present, 
                   // therefore the content is typcially used to document the option.  See examples
                   //
                   // If neither modifier is present the option is ccnsidered to be a switch, if the option 
                   // is present the value will be true, otherwise false, [def] if provided is ignored since 
                   // the default for this type ( not used ) will always be false 
                   
                   
           desc ,  // A description of the option that is printed out when --help or -h is used 
           
          [fn  ],  // An optional function that is passed the value and anything in the <> || [] modifier blocks
                   // The function can return a corece value to coninue, or an Error(msg) to terminate the 
                   // processing
                   
          [def ],  // A default value that will be used if the option is not required, ENV:name:def for this 
                   // parameter will extract the option from the 'name' in the enviorment if present
                   // or use the ':def' section if not, if ':def' section is not specified then it will be 
                   // undefined
        
```

  

```js
#!/usr/bin/env node

/**
 * Module dependencies.
 */

var program = require('commander');

program
  .version('0.0.1')
  .option('-p, --peppers', 'Add peppers')
  .option('-P, --pineapple', 'Add pineapple')
  .option('-b, --bbq', 'Add bbq sauce')
  .option('-c, --cheese [type]', 'Add the specified type of cheese [marble]', 'marble')
  .parse(process.argv);

console.log('you ordered a pizza with:');
if (program.peppers) console.log('  - peppers');
if (program.pineapple) console.log('  - pineapple');
if (program.bbq) console.log('  - bbq');
console.log('  - %s cheese', program.cheese);
```

 Short flags may be passed as a single arg, for example `-abc` is equivalent to `-a -b -c`. Multi-word options such as "--template-engine" are camel-cased, becoming `program.templateEngine` etc.

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

 You can display arbitrary `-h, --help` information
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
