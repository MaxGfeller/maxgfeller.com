---
layout: post
title: "composer test"
date: 2014-07-22 14:50
comments: true
categories:
---

Composer is a package manager for PHP which already serves
[~34'000 packages](http://modulecounts.com/) (as of July, 22nd) via
[Packagist](http://packagist.org) and grows with 70 new packages a day.

It is already used by the majority of open source PHP libraries, such as
[Symfony](https://github.com/symfony/symfony) and
[ZendFramework](https://github.com/zendframework/zf2).

However, when it comes to unit testing, most libraries demand you to have either
[PHPUnit](http://phpunit.de/) or [Behat](http://behat.org/) installed globally.

A change to composer that was [merged yesterday](https://github.com/composer/composer/pull/2516)
makes testing now as easy as running `composer test`.

## The `scripts` field

The `scripts` field in the `composer.json` was reserved for certain events until
now, such as `pre-package-install` or `post-update-cmd` which made it possible
to hook into these events and run additional scripts.

Additionally to this you can now define your own custom scripts and run them
with `composer run-script` or simply with `composer run`. Composer will also map
the `vendor/.bin` to the `$PATH` meaning that you can define `phpunit` or your
unit test framework of choice as a
[development dependency](https://getcomposer.org/doc/04-schema.md#require-dev):

```json
{
    "name": "example-app",
    "require-dev": {
        "phpunit/phpunit": "4.*"
    },
    "scripts": {
        "test": "phpunit -c app"
    }
}
```

Now you can simply run `composer test`.

## Additional tasks

Instead of `make` or `phing` you can define additional tasks directly in
your `composer.json` file now. So for example, if you need to provide a phar for
your app you could have a `composer.json` that looks like this:

```json
{
    "name": "example-app",
    "require-dev": {
        "kherge/box": "2.*"
    },
    "scripts": {
        "build": "box"
    }
}
```
Then you could package you app by running `composer run-script build`. This
would work because the `box` dependency provides an executable in the
`vendor/.bin` folder which is being mapped directly to the `$PATH`.
