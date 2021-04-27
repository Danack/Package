# Package

This is a really rough draft, just begin the conversation rather than a document designed to be persuasive.

## Introduction

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


It would be possible to have some shenanigans where *magic* happens on the fly to rename the classes to work around this.









