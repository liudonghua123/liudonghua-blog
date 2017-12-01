---
title: Web前端JS框架备忘
id: 99
categories:
  - 学习
date: 2014-10-14 20:25:08
tags:
---

桌面端最常用的有：[jQuery](https://jquery.com/ "jQuery is a fast, small, and feature-rich JavaScript library. It makes things like HTML document traversal and manipulation, event handling, animation, and Ajax much simpler with an easy-to-use API that works across a multitude of browsers. With a combination of versatility and extensibility, jQuery has changed the way that millions of people write JavaScript.")、[jQuery UI
](http://jqueryui.com/ "jQuery UI is a curated set of user interface interactions, effects, widgets, and themes built on top of the jQuery JavaScript Library. Whether you")移动端最常用的有：[jQuery Mobile](http://jquerymobile.com/ "jQuery Mobile is a HTML5-based user interface system designed to make responsive web sites and apps that are accessible on all smartphone, tablet and desktop devices.")<!--more-->

其中jQuery UI和jQuery Mobile都依赖于jQuery
jQuery分为1.x和2.x两个版本，区别是2.x不再支持IE 6\7\8

[zepto](http://zeptojs.com/ "Zepto is a minimalist JavaScript library for modern browsers with a largely jQuery-compatible API. If you use jQuery, you already know how to use Zepto.")

可以在桌面端和移动端都可以使用，兼容jQuery API(包含Core, Ajax, Event, Form, IE这5个jQuery模块)的为了减少体积及提高运行速度的库，其min版本不到jQuery min版本的1/3大小。

&nbsp;

[backbone](http://backbonejs.org/) ([Github](https://github.com/jashkenas/backbone))

[underscore](http://underscorejs.org/) ([Github](https://github.com/jashkenas/underscore))
Underscore is a JavaScript library that provides a whole mess of useful functional programming helpers without extending any built-in objects.
Underscore provides over 100 functions that support both your favorite workaday functional helpers: map, filter, invoke — as well as more specialized goodies: function binding, javascript templating, creating quick indexes, deep equality testing, and so on.

[differences-between-lodash-and-underscore](http://stackoverflow.com/questions/13789618/differences-between-lodash-and-underscore)

[lodash](https://lodash.com/) ([Github](https://github.com/lodash/lodash))
A utility library delivering consistency, customization, performance, &amp; extras.
是underscore的超集

## Features _not_ in Underscore

*   ~100% [code coverage](https://coveralls.io/r/lodash)
*   Module bundles for [AMD](https://github.com/lodash/lodash-amd/tree/2.4.1) &amp; [Node.js](https://npmjs.org/package/lodash-node) as well as [npm packages](https://npmjs.org/browse/keyword/lodash-modularized)
*   Follows [semantic versioning](http://semver.org/) for releases
*   [_(…)](https://lodash.com/docs#_) supports intuitive chaining
*   [_.at](https://lodash.com/docs#at) for cherry-picking collection values
*   [_.attempt](https://lodash.com/docs#attempt) to execute functions which may error without a try-catch
*   [_.before](https://lodash.com/docs#before) to complement [_.after](https://lodash.com/docs#after)
*   [_.bindKey](https://lodash.com/docs#bindKey) for binding [_“lazy”_](http://michaux.ca/articles/lazy-function-definition-pattern) defined methods
*   [_.chunk](https://lodash.com/docs#chunk) for splitting an array into chunks of a given size
*   [_.clone](https://lodash.com/docs#clone) supports shallow cloning of `Date` &amp; `RegExp` objects
*   [_.cloneDeep](https://lodash.com/docs#cloneDeep) for deep cloning arrays &amp; objects
*   [_.contains](https://lodash.com/docs#contains) accepts a `fromIndex`
*   [_.create](https://lodash.com/docs#create) for easier object inheritance
*   [_.curry](https://lodash.com/docs#curry) &amp; [_.curryRight](https://lodash.com/docs#curryRight) for creating [curried](http://hughfdjackson.com/javascript/why-curry-helps/) functions
*   [_.debounce](https://lodash.com/docs#debounce) &amp; [_.throttle](https://lodash.com/docs#throttle) are cancelable &amp; accept options for more control
*   [_.findIndex](https://lodash.com/docs#findIndex) &amp; [_.findKey](https://lodash.com/docs#findKey) for finding indexes &amp; keys
*   [_.flow](https://lodash.com/docs#flow) to complement [_.flowRight](https://lodash.com/docs#vlowRight) (a.k.a `_.compose`)
*   [_.forEach](https://lodash.com/docs#forEach) supports exiting early
*   [_.forIn](https://lodash.com/docs#forIn) for iterating all enumerable properties
*   [_.forOwn](https://lodash.com/docs#forOwn) for iterating own properties
*   [_.isError](https://lodash.com/docs#isError) to check for error objects
*   [_.isNative](https://lodash.com/docs#isNative) to check for native functions
*   [_.isPlainObject](https://lodash.com/docs#isPlainObject) to check for objects created by `Object`
*   [_.keysIn](https://lodash.com/docs#keysIn) &amp; [_.valuesIn](https://lodash.com/docs#valuesIn) for getting keys and values of all enumerable properties
*   [_.mapValues](https://lodash.com/docs#mapValues) for [mapping](https://lodash.com/docs#map) values to an object
*   [_.merge](https://lodash.com/docs#merge) for a deep [_.extend](https://lodash.com/docs#extend)
*   [_.parseInt](https://lodash.com/docs#parseInt) for consistent behavior
*   [_.pull](https://lodash.com/docs#pull), [_.pullAt](https://lodash.com/docs#pullAt), &amp; [_.remove](https://lodash.com/docs#remove) for mutating arrays
*   [_.random](https://lodash.com/docs#random) supports returning floating-point numbers
*   [_.runInContext](https://lodash.com/docs#runInContext) for collisionless mixins &amp; easier mocking
*   [_.slice](https://lodash.com/docs#slice) for creating subsets of array-like values
*   [_.sortBy](https://lodash.com/docs#sortBy) supports sorting by multiple properties
*   [_.support](https://lodash.com/docs#support) for flagging environment features
*   [_.template](https://lodash.com/docs#template) supports [_“imports”_](https://lodash.com/docs#templateSettings_imports) options &amp; [ES6 template delimiters](http://people.mozilla.org/~jorendorff/es6-draft.html#sec-template-literal-lexical-components)
*   [_.transform](https://lodash.com/docs#transform) as a powerful alternative to [_.reduce](https://lodash.com/docs#reduce) for transforming objects
*   [_.thru](https://lodash.com/docs#thru) to pass values thru method chains
*   [_.where](https://lodash.com/docs#where) supports deep object comparisons
*   [_.xor](https://lodash.com/docs#xor) to complement [_.difference](https://lodash.com/docs#difference), [_.intersection](https://lodash.com/docs#intersection), &amp; [_.union](https://lodash.com/docs#union)
*   [_.bind](https://lodash.com/docs#bind), [_.curry](https://lodash.com/docs#curry), [_.partial](https://lodash.com/docs#partial), &amp; [more](https://lodash.com/docs "_.bindKey, _.curryRight, _.partialRight") support argument placeholders
*   [_.capitalize](https://lodash.com/docs#capitalize), [_.trim](https://lodash.com/docs#trim), &amp; [more](https://lodash.com/docs "_.camelCase, _.deburr, _.endsWith, _.escapeRegExp, _.kebabCase, _.pad, _.padLeft, _.padRight, _.repeat, _.snakeCase, _.startsWith, _.trimLeft, _.trimRight, _.trunc, _.words") string methods
*   [_.clone](https://lodash.com/docs#clone), [_.isEqual](https://lodash.com/docs#isEqual), &amp; [more](https://lodash.com/docs "_.assign, _.cloneDeep, _.merge") accept callbacks
*   [_.contains](https://lodash.com/docs#contains), [_.toArray](https://lodash.com/docs#toArray), &amp; [more](https://lodash.com/docs "_.at, _.countBy, _.every, _.filter, _.find, _.findLast, _.forEach, _.forEachRight, _.groupBy, _.indexBy, _.invoke, _.map, _.max, _.min, _.partition, _.pluck, _.reduce, _.reduceRight, _.reject, _.shuffle, _.size, _.some, _.sortBy") accept strings
*   [_.dropWhile](https://lodash.com/docs#dropWhile), [_.takeWhile](https://lodash.com/docs#takeWhile), &amp; [more](https://lodash.com/docs "_.drop, _.dropRightWhile, _.take, _.takeRightWhile") to complement [_.first](https://lodash.com/docs#first), [_.initial](https://lodash.com/docs#initial), [_.last](https://lodash.com/docs#last), &amp; [_.rest](https://lodash.com/docs#rest)
*   [_.findLast](https://lodash.com/docs#findLast), [_.findLastIndex](https://lodash.com/docs#findLastIndex), &amp; [more](https://lodash.com/docs "_.findLastKey, _.flowRight, _.forEachRight, _.forInRight, _.forOwnRight, _.partialRight") right-associative methods