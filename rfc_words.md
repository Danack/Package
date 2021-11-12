# Package

This is a really rough draft, just begin the conversation rather than a document designed to be persuasive.

## Introduction

Currently the way that libraries are used in PHP just isn't as nice as in JavaScript. There's a couple of overlapping problems, such as only one version of a library being installable at once, having to pull in functions requires a giant list of use statements, and that PECL is a massive impediment.

Being able to declare that all files that are part of a package, should have a particular set of functions imported and available in all of those files, would make writing code quite a bit easier. probably.

## Basic code examples

Rather than writing many words, have some code examples that demonstrate package ideas and uses.

### Package declaration is first line in code.

```php
<?php

package Demo;

...

```

Any file using package delcaration cannot use any declare statement.

### Packages are registerable from code

```php
<?php

$options = [
  'compile' => [
    'strictness' => 'strict'
  ]
];

package_register('Demo', $options);
```

### Unknown options are returned

To allow code to run on multiple PHP versions, unknown options do not give an error, but instead are returned:

```php
<?php

$options = [
  'compile' => [
    'strictness' => 'strict'
  ],
  'some_option_introducedin_php_9' => true
];

$result = package_register('Demo', $options);
var_dump($result);
// array([
// 'unknown options' => ['some_option_introducedin_php_9' => true] 
//])
```

This allows the code registering the package to figure out what to do for unsupported options.

### Using unknown package gives a compile error

```php
<?php

package UnknownPackage;

```

Include'ing or require'ing this file results in a compile error 'UnknownPackage'


## More possibilities

I think just the above would probably useful enough to consider doing, but there are many more features that would be easier to implement through the general package concept.


### Private classes

There have been quite a few discussions over the years where


```php

$options = [
    'access' => [
        '\Demo\SomePrivateNamespace' => [
            // empty array, classes in this namespace can only be instantiated,
            // extended or implemented in this namespace
        ],
        '\Demo\SomeProtectedNamespace' => [
            '\Demo\SomePublicNamespace'
            // Any class in the namespace Demo\SomeProtectedNamespace can only be
            // instantiated, extended or implemented from either the 
            // \Demo\SomeProtectedNamespace or \Demo\SomePublicNamespace namespace
        ],
    ],
];

```


### More compile options

(this is where people start shouting at me).

#### Automatically adding traits

The default behaviour around __get and __set has annoyed me. I usually try to remember to add a SafeAccess trait to give a clear error message, but doing it per class is tedious. 

```php
<?php
$options = [
    'compile' => [
        'autotraits' => [
            'Demo\SomeNamespace' => [
                'Danack\SafeAccess'
                // Any class in the namespace Demo\SomeNamespace automatically
                // has the trait Danack\SafeAccess use'd in it.  
            ]
        ]
    ],
];

```


#### Default importing functions and classes namespaces

Having huge blocks of use statements is annoying, as is the alternative of having to use full namespace paths.

```php
<?php
$options = [
  'compile' => [
    'use_functions' => [
      'SomeNamespace' => [
        'Foo\bar'  
      ]
    ]
  ],
];

package_register('Demo', $options);
```

The currently possible code of:
```php
<?php

package Demo;

namespace SomeNamespace;

use function \Foo\bar;

bar();
```

Would be equivalent to:
```php
<?php

package Demo;

namespace SomeNamespace;

bar();
```


#### Remapping namespaces

One limitation in PHP currently is that one version of the a library will clash with a different version of the same library.

That isn't too bad for small applications, but gets annoying where multiple of your dependencies use the same library and for various reasons one of them might have a non-compatible verison range requirement with another one.

It would be possible to have some shenanigans where *vast quantities of magic* happens on the fly to rename the classes to work around this.



## Why this is a good idea

### Easier to change PHP

Some changes to the PHP language are hard to carry out in a way that has a good developer experience for PHP users.

For example [foreach unwrap ref](https://wiki.php.net/rfc/foreach_unwrap_ref) is probably a good idea but users can't easily tell if it will affect them (without static analysis tools) without first upgrading the whole of their application to work on that version of PHP.

Having to do all of the upgrade work in a single large chunk is difficult as if you find bugs start happening in production you have to roll back to both the previous version of PHP and the application code before the upgrade.

Being able to separate upgrading to a new version of PHP and opting-in to the new feature/compile settings would allow easier upgrades. The following walks through some example scenarios for how it could work.

#### Example of fast path

Imagine that we already have per-package compile settings. Here is a table of:
* the default value of the foreach_unwrap_ref setting
* whether users are required to set it or not
* whether it's a recognised option or not

|      | Default | Required | Recognised  |
|------|---------|----------|-------------|
|  8.2 | false   | No       | Yes         |
|  8.3 | true    | Yes      | Yes         |
|  8.4 | true    | No       | Yes         |
|  8.5 | true    | No       | No          |

At every step users code can be configured to run safely on two versions of PHP. People who are keen to optin can do so on PHP 8.2. Users who want to wait can wait until PHP 8.3 by setting the option to false.

Any user who had forgotten/missed that foreach_unwrap_ref needed checking would get an error when running their code on PHP 8.3, rather than risking silent changed before.


#### Example of slow path

We could also slow the upgrade process down a bit, by tweaking which versions the setting is required to be set on:

Here is a table of:
* the default value of the foreach_unwrap_ref setting
* whether users are required to set it or not
* whether it's a recognised option or not


|      | Default | Required | Recognised  |
|------|---------|----------|-------------|
|  8.2 | false   | No       | Yes         |
|  8.3 | false   | Yes      | Yes         |
|  8.4 | true    | Yes      | Yes         |
|  8.5 | true    | No       | Yes         |
|  8.6 | true    | No       | No          |

At every step users code can be configured to run safely on at least two versions of PHP, and sometimes 3, that would give people a longer time to upgrade their apps.


#### Example of super fast path

It would also be possible to force people to upgrade their apps faster, by having no version where the setting is required to be set.


|      | Default | Required | Recognised  |
|------|---------|----------|-------------|
|  8.2 | false   | No       | Yes         |
|  8.3 | true    | No       | Yes         |
|  8.4 | true    | No       | No          |

This is the same as 'fast path' except that there isn't a version that alerts users to check that a option is changing. This means that some people wouldn't notice that a setting had changed.

#### Upgrading applications piecemeal

As users could set compile options per package, and can use as many different packages in their application, that would allow them to upgrade their application a piece at a time.

Being able to do upgrades in smaller pieces of work is much easier and less risk. If something starts having bugs in production, they can just roll back that small change. That is much better than having to roll back to a previous version of PHP, and all of the changes made to their app.
