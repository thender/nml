nml
===
[![Build Status](https://travis-ci.org/maddenp/nml.svg)](https://travis-ci.org/maddenp/nml)

A query/modify utility for Fortran namelists

###Build

Install [Leiningen](http://leiningen.org/) if you don't have it, then:

`lein uberjar`

###Run

The _nml_ wrapper script invokes _java -jar_ with the path to the Leiningen-generated _target/nml.jar_. It may be convenient to edit this script for your own use.

````
Usage: nml [options]

Options:

  -c, --create      Create new namelist
  -e, --edit file   Edit file (instead of '-i file -o file')
  -f, --format fmt  Output in format 'fmt' (default: namelist)
  -g, --get n:k     Get value of key 'k' in namelist 'n'
  -h, --help        Show usage information
  -i, --in file     Input file (default: stdin)
  -n, --no-prefix   Report values without 'namelist:key=' prefix
  -o, --out file    Output file (default: stdout)
  -s, --set n:k=v   Set value of key 'k' in namelist 'n' to 'v'
  -v, --version     Show version information

Valid output formats are: bash, ksh, namelist
````

###Examples

Assume that the contents of the file _nl_ are as follows:

```
&b
  L = .TRUE. ! logical value
  c=( 3.142 , 2.718)
  ! This is a comment.
  i=88
/

&a r=1.1e8, S = 'Hello World' r=2.2e8 /

This junk isn't in a namelist.
```

By default, _nml_ reads from stdin and writes to stdout and, with no command-line options specified, prints a simplified, sorted version of the input:

```
% cat nl | nml
&a
  r=2.2e8
  s='Hello World'
/
&b
  c=(3.142,2.718)
  i=88
  l=t
/
````

Note that _nml_ normalizes many formatting options: Whitespace and comments are removed, non-string text is converted to lower-case, key-value pairs are printed one-per-line without comma separators, logical values are represented in their simplest form, etc.

The _--in_ and _--out_ options can be used to specify input and output files, respectively.

#####Querying

To get values:

````
% nml --in nl --get a:r --get b:i
a:r=2.2e8
b:i=88
````

To print only values, without _namelist:key=_ prefixes:

````
% nml --in nl --no-prefix --get b:i --get a:r
88
2.2e8
````

Note that values are printed in the order they were requested on the command line.

An obvious application is to use _nml_ to insert namelist settings in scripts:

```
% cat say.sh
#!/bin/sh
echo "The value of i is $(nml --in nl --no-prefix --get b:i)"

% ./say.sh
The value of i is 88
````

If any requested namelists or keys are not found, _nml_ reports the first ungettable value and exits with error status:

````
% nml --in nl --get a:r --get b:x || echo "FAIL"
nml: b:x not found
FAIL
````

#####Modifying

To set (or add) values:

````
% nml --in nl --set a:s="'Hi'" --set b:x=.false. --set c:z=99
&a
  r=2.2e8
  s='Hi'
/
&b
  c=(3.142,2.718)
  i=88
  l=t
  x=f
/
&c
  z=99
/
````

Note that get and set commands may not be mixed in a single _nml_ invovation.

A file may be edited in place with the _--edit_ command (equivalent, in this case, to _--in nl --out nl_):

````
% nml --in nl --get a:s
a:s='Hello World'
% nml --edit nl --set a:s="'Hi'"
% nml --in nl --get a:s
a:s='Hi'
````

To create a new namelist file from scratch (i.e. without starting with an input file):

````
% rm -f new

% nml --create --out new --set a:x=77 --set a:y=88

% cat new
&a
  x=77
  y=88
/
````

#####Output Format

In addition to the default Fortran namelist output format, _nml_ can output namelist data as a bash/ksh function. This allows fast lookups in shell scripts after a single _nml_ invocation, via the defined _nmlquery_ shell function.

````
% eval "$(nml --in nl --format bash)"

% nmlquery a s
"Hi"

% nmlquery b c
(3.142,2.718)
````

###Thanks

Thanks to Mark Engelberg for the wonderful [Instaparse](https://github.com/Engelberg/instaparse), on which _nml_ is based.
