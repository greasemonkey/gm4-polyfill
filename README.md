# gm4-polyfill.js

This helper script bridges compatibility between the Greasemonkey 4 (GM4) APIs and Greasemonkey 3 (GM3) APIs (as exist in GM3, Tampermonkey (TM), ViolentMonkey (VM), etc.).  It is intended for use in scripts which use the GM4 APIs to allow the script to function without changes in Greasemonkey 3, or other user-script managers. It can be included in your script using:

    // @require https://greasemonkey.github.io/gm4-polyfill/gm4-polyfill.js

If your script is running in an engine that does not provide the new GM4 asynchronous APIs, this helper will add them, based on the old APIs. It does this by wrapping each GM3 API in a [`Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

## Updating a script from Greasemonkey 3 APIs (GM-dash) to Greasemonkey 4 APIs (GM-dot):

Say, for example, your Greasemonkey 3 user-script includes:

    // @grant   GM_getValue

And you'd like to be compatible with both Greasemonkey 4 and Greasemonkey 3 (and for that matter all versions of Tampermonkey, Violentmonkey, and any other user-script engine).  Add:

    // @grant   GM.getValue
    // @require https://greasemonkey.github.io/gm4-polyfill/gm4-polyfill.js

Thus, you will have all three lines in your script's [Metadata Block](https://wiki.greasespot.net/Metadata_Block):

    // @grant   GM_getValue
    // @grant   GM.getValue
    // @require https://greasemonkey.github.io/gm4-polyfill/gm4-polyfill.js

Then, change your script to call the new GM-dot APIs (`GM.*`), which return [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

Using normal Promise syntax, you can call `GM.getValue()` like:

    (() => {
      GM.getValue('x').then(x => console.log(x));
    })();



### Using [`async`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function) and [`await`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
One easy way to change to using the asynchronous APIs is to use `await`, which leaves the code structure similar to using a synchronous call. To use `await`, you must declare the function containing the call to the GM4 API as an `async` function and then `await` the result from the API. You must to do this for all functions in the call-chain which are written with the expectation that the called function is fully synchronous.

If you use `await` at the top level, you'll need to wrap your script in an `async` function in order to semantically treat the `await` call as if it is synchronous. Using `async` and `await`, the `GM.getValue()` code above could be written as:

    (async () => {
      let x = await GM.getValue('x');
      console.log(x);
    })();

#### Browser compatibility for `async` and `await`
Browser support for Promises, which are widely supported, is inherently required for this polyfill. Using `async` and `await` is merely a convenient syntax for writing asynchronous code using Promises with a semantic structure similar to synchronous code. This allows changing code written with calls to the synchronous GM3 APIs without the need to restructure to handle the asynchronous nature of the GM-dot APIs.

Most browsers support `async` and `await`, but they're not available in Pale Moon as of the current version, 27.6.2. Thus, to support the current version of Pale Moon, you will need to restructure the code to use the standard Promise syntax (e.g. [`.then()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then)).

## Examples
### Using `GM.setValue()` and `GM.getValue()`

The following example, demonstrating `GM.setValue()` and `GM.getValue()`, will work in Greasemonkey 4, Greasemonkey 3, Tampermonkey, Violentmonkey, etc.

    // ==UserScript==
    // @name        Test gm4-polyfill getValue and setValue
    // @namespace   gm4-polyfill-test
    // @description Test gm4-polyfill.js @require'd by a script using getValue() and setValue().
    // @version     1.0.0
    // @match       *://*/*
    // @grant       GM.getValue
    // @grant       GM.setValue
    // @grant       GM_getValue
    // @grant       GM_setValue
    // @require     https://greasemonkey.github.io/gm4-polyfill/gm4-polyfill.js
    // ==/UserScript==

    (() => {
      'use strict';

      console.log('GM:', GM);

      let random = Math.random();
      console.log('random: ', random);

      GM.setValue('testValue', random).then(() => {
        GM.getValue('testValue').then(value => {
          console.log('value: ', value);
          // Edge/TM requires the < to be quoted \<
          console.log('Stored value === retrieved value:', value === random,
            (value === random ? '-->PASS\<--' : '-->FAIL\<--'));
        });
      });
    })();


### Using `GM.info` with `@grant none`

The following example, demonstrating `GM.info` with `@grant none`, will work in Greasemonkey 4, Greasemonkey 3, Tampermonkey, Violentmonkey, etc.

    // ==UserScript==
    // @name        Test gm4-polyfill with grant none
    // @namespace   gm4-polyfill-test
    // @description Test gm4-polyfill.js @require'd by a script with @grant none.
    // @version     1.0.0
    // @match       *://*/*
    // @grant       none
    // @require     https://greasemonkey.github.io/gm4-polyfill/gm4-polyfill.js
    // ==/UserScript==

    (() => {
      'use strict';

      console.log('GM:', GM);
      console.log('GM.info:', GM.info);
    })();


## Target environments
The intent is that *gm4-polyfill.js* is used to allow developers to have one user-script which is compatible with the user-script APIs commonly available in all operating environments (operating system / browser / user-script manager). The goal is that `@require` can be used to include *gm4-polyfill.js* in user-scripts intended for all operating environments.     

The following is a *non-exhaustive* list of environments in which the polyfill is intended, and tested, to function. The desire is to be inclusive of operating environments, not exclusive. The list is intended to demonstrate and inform that the script needs to be functional in a wide variety of environments. This list is not intended to restrict operation to only this set of environments.

Abbreviations: Greasemonkey 4 (GM4), Greasemonkey 3.17 (GM3), Tampermonkey (TM), and Violentmonkey (VM).
### Base browsers / user-script manager combinations
* Firefox 57+: GM4, TM, VM
* Firefox 56.0.2: GM4, GM3, TM, VM
* Firefox ESR 52 (currently: 52.5.2)<sup>1</sup>: GM3, TM, VM
* Chrome 60+ (currently: 63.0.3239.84): TM, VM
* Opera 49+ (currently: 49.0.2725.64 (PGO)): TM
* Pale Moon (currently: 27.6.2): Greasemonkey for Pale Moon 3.30rc4

<sup>1. Firefox ESR 52: [GM4: `GM.setValue` and `GM.getValue()` are broken in Firefox ESR 52.5.2](https://github.com/greasemonkey/greasemonkey/issues/2781).</sup>

### Operating systems (with additional browsers / user-script managers)
* Linux (multiple releases)   <!-- Please don't add an exhaustive list of Linux builds/releases. There are just too many. -->
* macOS (OS X)
  * Safari: TM
* Windows 7 through Windows 10
  * Edge 41+ (currently: 41.16299.15.0): TM

## Contributing
### Test prior to submitting a pull request
Prior to submitting a pull request (PR), **test your changes**. Changes to this polyfill affect all scripts which `@require` it. As a result, your change can affect a *very wide* audience. Any issues with changes to the script can cause ongoing problems, which may not be immediately evident. Code included in a user-script with `@require` is not downloaded on each page load. It is downloaded once and rarely updated. Thus, changes to this script may take time to propagate to users. Versions with errors may persist on user machines for quite some time. So, make sure your changes are working prior to submitting a PR.

The amount of testing needed to cover the entire spectrum of operating systems / browsers / user-script managers is huge. It is understandable that people will want to test on a limited subset.

#### At a *minimum*:
* Use both of the above example scripts. The user-script environment is different between `@grant none` and when one of the APIs is granted.
* Test on (at least) example.com and github.com (e.g. this page). GitHub uses a restrictive [CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) setting, which makes for a different environment, particularly for `@grant none`.
* Use whichever operating system is your "native" OS.
* Test in any additional browser / user-script manager listed above in [Target environments - Operating systems](#user-content-operating-systems-with-additional-browsers--user-script-managers) for your "native" OS.
* Test in the following combinations of browsers / user-script managers. If a specific version is not listed, use the currently released version of each:
  * Firefox 57+: GM4, TM, VM
  * Firefox 56.0.2: GM4, GM3, TM, VM
  * Firefox ESR 52: GM3, TM, VM
  * Chrome 63+: TM, VM

More  extensive testing is desirable. There are [quirks](https://github.com/greasemonkey/gm4-polyfill/pull/9) in various different combinations.

In your PR, state what testing you did and the environments in which you tested.

### Version numbers
Version numbers in this repository follow [Semantic Versioning 2.0.0](https://semver.org/). If the gm4-polyfill.js file changes, then the version number should change (once per PR or merge). If you change either of the above examples, then their version number(s) should change.

### Licensing
By contributing to this repository you agree to have your contribution distributed under the MIT license as currently applied to this repository.
