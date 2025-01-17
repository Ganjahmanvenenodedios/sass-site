---
title: sass:meta
---

{% render 'doc_snippets/built-in-module-status' %}

## Mixins

{% function 'meta.load-css($url, $with: null)' %}
  {% compatibility 'dart: "1.23.0"', 'libsass: false', 'ruby: false' %}
    Only Dart Sass currently supports this mixin.
  {% endcompatibility %}

  Loads the [module][] at `$url` and includes its CSS as though it were written
  as the contents of this mixin. The `$with` parameter provides
  [configuration][] for the modules; if it's passed, it must be a map from
  variable names (without `$`) to the values of those variables to use in the
  loaded module.

  [module]: /documentation/at-rules/use
  [configuration]: /documentation/at-rules/use#configuration

  If `$url` is relative, it's interpreted as relative to the file in which
  `meta.load-css()` is included.

  **Like the [`@use` rule][]**:

  [`@use` rule]: /documentation/at-rules/use

  * This will only evaluate the given module once, even if it's loaded multiple
    times in different ways.

  * This cannot provide configuration to a module that's already been loaded,
    whether or not it was already loaded with configuration.

  **Unlike the [`@use` rule][]**:

  * This doesn't make any members from the loaded module available in the
    current module.

  * This can be used anywhere in a stylesheet. It can even be nested within
    style rules to create nested styles!

  * The module URL being loaded can come from a variable and include
    [interpolation][].

    [interpolation]: /documentation/interpolation

  {% headsUp %}
    The `$url` parameter should be a string containing a URL like you'd pass to
    the `@use` rule. It shouldn't be a CSS `url()`!
  {% endheadsUp %}

  {% codeExample 'load-css', false %}
    // dark-theme/_code.scss
    $border-contrast: false !default;

    code {
      background-color: #6b717f;
      color: #d2e1dd;
      @if $border-contrast {
        border-color: #dadbdf;
      }
    }
    ---
    // style.scss
    @use "sass:meta";

    body.dark {
      @include meta.load-css("dark-theme/code",
          $with: ("border-contrast": true));
    }
    ===
    // dark-theme/_code.sass
    $border-contrast: false !default

    code
      background-color: #6b717f
      color: #d2e1dd
      @if $border-contrast
        border-color: #dadbdf
    ---
    // style.sass
    @use "sass:meta"

    body.dark
      $configuration: ("border-contrast": true)
      @include meta.load-css("dark-theme/code", $with: $configuration)
    ===
    body.dark code {
      background-color: #6b717f;
      color: #d2e1dd;
      border-color: #dadbdf;
    }
  {% endcodeExample %}
{% endfunction %}

## Functions

{% function 'meta.calc-args($calc)', 'returns:list' %}
  {% compatibility 'dart: "1.40.0"', 'libsass: false', 'ruby: false' %}{% endcompatibility %}

  Returns the arguments for the given [calculation].

  [calculation]: /documentation/values/calculations

  If an argument is a number or a nested calculation, it's returned as that
  type. Otherwise, it's returned as an unquoted string.

  {% codeExample 'calc-args', false %}
    @debug meta.calc-args(calc(100px + 10%)); // unquote("100px + 10%")
    @debug meta.calc-args(clamp(50px, var(--width), 1000px)); // 50px, unquote("var(--width)"), 1000px
    ===
    @debug meta.calc-args(calc(100px + 10%))  // unquote("100px + 10%")
    @debug meta.calc-args(clamp(50px, var(--width), 1000px))  // 50px, unquote("var(--width)"), 1000px
  {% endcodeExample %}
{% endfunction %}

{% function 'meta.calc-name($calc)', 'returns:quoted string' %}
  {% compatibility 'dart: "1.40.0"', 'libsass: false', 'ruby: false' %}{% endcompatibility %}

  Returns the name of the given [calculation].

  [calculation]: /documentation/values/calculations

  {% codeExample 'calc-name', false %}
    @debug meta.calc-name(calc(100px + 10%)); // "calc"
    @debug meta.calc-name(clamp(50px, var(--width), 1000px)); // "clamp"
    ===
    @debug meta.calc-name(calc(100px + 10%))  // "calc"
    @debug meta.calc-name(clamp(50px, var(--width), 1000px))  // "clamp"
  {% endcodeExample %}
{% endfunction %}

{% function 'meta.call($function, $args...)', 'call($function, $args...)' %}
  {% render 'doc_snippets/call-impl-status' %}

  Invokes `$function` with `$args` and returns the result.

  The `$function` should be a [function][] returned by
  [`meta.get-function()`][].

  [function]: /documentation/values/functions
  [`meta.get-function()`]: #get-function

  {% render 'code_snippets/example-first-class-function' %}
{% endfunction %}

{% function 'meta.content-exists()', 'content-exists()', 'returns:boolean' %}
  Returns whether the current mixin was passed a [`@content` block][].

  [`@content` block]: /documentation/at-rules/mixin#content-blocks

  Throws an error if called outside of a mixin.

  {% codeExample 'content-exists', false %}
    @mixin debug-content-exists {
      @debug meta.content-exists();
      @content;
    }

    @include debug-content-exists; // false
    @include debug-content-exists { // true
      // Content!
    }
    ===
    @mixin debug-content-exists
      @debug meta.content-exists()
      @content


    @include debug-content-exists  // false
    @include debug-content-exists   // true
      // Content!
  {% endcodeExample %}
{% endfunction %}

{% function 'meta.feature-exists($feature)', 'feature-exists($feature)', 'returns:boolean' %}
  Returns whether the current Sass implementation supports `$feature`.

  The `$feature` must be a string. The currently recognized features are:

  * `global-variable-shadowing`, which means that a local variable will
    [shadow][] a global variable unless it has the `!global` flag.
  * `extend-selector-pseudoclass`, which means that the [`@extend` rule][] will
    affect selectors nested in pseudo-classes like `:not()`.
  * `units-level3`, which means that [unit arithmetic][] supports units defined
    in [CSS Values and Units Level 3][].
  * `at-error`, which means that the [`@error` rule][] is supported.
  * `custom-property`, which means that [custom property declaration][] values
    don't support any [expressions][] other than [interpolation][].

  [shadow]: /documentation/variables#shadowing
  [`@extend` rule]: /documentation/at-rules/extend
  [unit arithmetic]: /documentation/values/numbers#units
  [CSS Values and Units Level 3]: http://www.w3.org/TR/css3-values
  [`@error` rule]: /documentation/at-rules/error
  [custom property declaration]: /documentation/style-rules/declarations#custom-properties
  [expressions]: /documentation/syntax/structure#expressions
  [interpolation]: /documentation/interpolation

  Returns `false` for any unrecognized `$feature`.

  {% codeExample 'feature-exists', false %}
    @debug meta.feature-exists("at-error"); // true
    @debug meta.feature-exists("unrecognized"); // false
    ===
    @debug meta.feature-exists("at-error")  // true
    @debug meta.feature-exists("unrecognized")  // false
  {% endcodeExample %}
{% endfunction %}

{% function 'meta.function-exists($name, $module: null)', 'function-exists($name)', 'returns:boolean' %}
  Returns whether a function named `$name` is defined, either as a built-in
  function or a user-defined function.

  If `$module` is passed, this also checks the module named `$module` for the
  function definition. `$module` must be a string matching the namespace of a
  [`@use` rule][] in the current file.

  [`@use` rule]: /documentation/at-rules/use

  {% codeExample 'function-exists', false %}
    @use "sass:math";

    @debug meta.function-exists("div", "math"); // true
    @debug meta.function-exists("scale-color"); // true
    @debug meta.function-exists("add"); // false

    @function add($num1, $num2) {
      @return $num1 + $num2;
    }
    @debug meta.function-exists("add"); // true
    ===
    @use "sass:math"

    @debug meta.function-exists("div", "math")  // true
    @debug meta.function-exists("scale-color")  // true
    @debug meta.function-exists("add")  // false

    @function add($num1, $num2)
      @return $num1 + $num2

    @debug meta.function-exists("add")  // true
  {% endcodeExample %}
{% endfunction %}

{% function 'meta.get-function($name, $css: false, $module: null)', 'get-function($name, $css: false, $module: null)', 'returns:function' %}
  Returns the [function][] named `$name`.

  [function]: /documentation/values/functions

  If `$module` is `null`, this returns the function named `$name` without a
  namespace (including [global built-in functions][]). Otherwise, `$module` must
  be a string matching the namespace of a [`@use` rule][] in the current file,
  in which case this returns the function in that module named `$name`.

  [global built-in functions]: /documentation/modules#global-functions
  [`@use` rule]: /documentation/at-rules/use

  By default, this throws an error if `$name` doesn't refer to Sass function.
  However, if `$css` is `true`, it instead returns a [plain CSS function][].

  [user-defined]: /documentation/at-rules/function
  [plain CSS function]: /documentation/at-rules/function/#plain-css-functions

  The returned function can be called using [`meta.call()`](#call).

  {% render 'code_snippets/example-first-class-function' %}
{% endfunction %}

{% function 'meta.global-variable-exists($name, $module: null)', 'global-variable-exists($name, $module: null)', 'returns:boolean' %}
  Returns whether a [global variable][] named `$name` (without the `$`) exists.

  [global variable]: /documentation/variables#scope

  If `$module` is `null`, this returns whether a variable named `$name` without
  a namespace exists. Otherwise, `$module` must be a string matching the
  namespace of a [`@use` rule][] in the current file, in which case this returns
  whether that module has a variable named `$name`.

  [`@use` rule]: /documentation/at-rules/use

  See also [`meta.variable-exists()`](#variable-exists).

  {% codeExample 'global-variable-exists', false %}
    @debug meta.global-variable-exists("var1"); // false

    $var1: value;
    @debug meta.global-variable-exists("var1"); // true

    h1 {
      // $var2 is local.
      $var2: value;
      @debug meta.global-variable-exists("var2"); // false
    }
    ===
    @debug meta.global-variable-exists("var1")  // false

    $var1: value
    @debug meta.global-variable-exists("var1")  // true

    h1
      // $var2 is local.
      $var2: value
      @debug meta.global-variable-exists("var2")  // false
  {% endcodeExample %}
{% endfunction %}

{% function 'meta.inspect($value)', 'inspect($value)', 'returns:unquoted string' %}
  Returns a string representation of `$value`.

  Returns a representation of *any* Sass value, not just those that can be
  represented in CSS. As such, its return value is not guaranteed to be valid
  CSS.

  {% headsUp %}
    This function is intended for debugging; its output format is not guaranteed
    to be consistent across Sass versions or implementations.
  {% endheadsUp %}

  {% codeExample 'inspect', false %}
    @debug meta.inspect(10px 20px 30px); // unquote("10px 20px 30px")
    @debug meta.inspect(("width": 200px)); // unquote('("width": 200px)')
    @debug meta.inspect(null); // unquote("null")
    @debug meta.inspect("Helvetica"); // unquote('"Helvetica"')
    ===
    @debug meta.inspect(10px 20px 30px)  // unquote("10px 20px 30px")
    @debug meta.inspect(("width": 200px))  // unquote('("width": 200px)')
    @debug meta.inspect(null)  // unquote("null")
    @debug meta.inspect("Helvetica")  // unquote('"Helvetica"')
  {% endcodeExample %}
{% endfunction %}

{% function 'meta.keywords($args)', 'keywords($args)', 'returns:map' %}
  Returns the keywords passed to a mixin or function that takes [arbitrary
  arguments][]. The `$args` argument must be an [argument list][].

  [arbitrary arguments]: /documentation/at-rules/mixin#taking-arbitrary-arguments
  [argument list]: /documentation/values/lists#argument-lists

  The keywords are returned as a map from argument names as unquoted strings
  (not including `$`) to the values of those arguments.

  {% render 'code_snippets/example-mixin-arbitrary-keyword-arguments' %}
{% endfunction %}

{% function 'meta.mixin-exists($name, $module: null)', 'mixin-exists($name, $module: null)', 'returns:boolean' %}
  Returns whether a [mixin][] named `$name` exists.

  [mixin]: /documentation/at-rules/mixin

  If `$module` is `null`, this returns whether a mixin named `$name` without a
  namespace exists. Otherwise, `$module` must be a string matching the namespace
  of a [`@use` rule][] in the current file, in which case this returns whether
  that module has a mixin named `$name`.

  [`@use` rule]: /documentation/at-rules/use

  {% codeExample 'mixin-exists', false %}
    @debug meta.mixin-exists("shadow-none"); // false

    @mixin shadow-none {
      box-shadow: none;
    }

    @debug meta.mixin-exists("shadow-none"); // true
    ===
    @debug meta.mixin-exists("shadow-none")  // false

    @mixin shadow-none
      box-shadow: none


    @debug meta.mixin-exists("shadow-none")  // true
  {% endcodeExample %}
{% endfunction %}

{% function 'meta.module-functions($module)', 'returns:map' %}
  {% render 'doc_snippets/module-system-function-status' %}

  Returns all the functions defined in a module, as a map from function names to
  [function values][].

  [function values]: /documentation/values/functions

  The `$module` parameter must be a string matching the namespace of a [`@use`
  rule][] in the current file.

  [`@use` rule]: /documentation/at-rules/use

  {% codeExample 'module-functions', false %}
    // _functions.scss
    @function pow($base, $exponent) {
      $result: 1;
      @for $_ from 1 through $exponent {
        $result: $result * $base;
      }
      @return $result;
    }
    ---
    @use "sass:map";
    @use "sass:meta";

    @use "functions";

    @debug meta.module-functions("functions"); // ("pow": get-function("pow"))

    @debug meta.call(map.get(meta.module-functions("functions"), "pow"), 3, 4); // 81
    ===
    // _functions.sass
    @function pow($base, $exponent)
      $result: 1
      @for $_ from 1 through $exponent
        $result: $result * $base

      @return $result
    ---
    @use "sass:map"
    @use "sass:meta"

    @use "functions"

    @debug meta.module-functions("functions") // ("pow": get-function("pow"))

    @debug meta.call(map.get(meta.module-functions("functions"), "pow"), 3, 4) // 81
  {% endcodeExample %}
{% endfunction %}

{% function 'meta.module-variables($module)', 'returns:map' %}
  {% render 'doc_snippets/module-system-function-status' %}

  Returns all the variables defined in a module, as a map from variable names
  (without `$`) to the values of those variables.

  The `$module` parameter must be a string matching the namespace of a [`@use`
  rule][] in the current file.

  [`@use` rule]: /documentation/at-rules/use

  {% codeExample 'module-variables', false %}
    // _variables.scss
    $hopbush: #c69;
    $midnight-blue: #036;
    $wafer: #e1d7d2;
    ---
    @use "sass:meta";

    @use "variables";

    @debug meta.module-variables("variables");
    // (
    //   "hopbush": #c69,
    //   "midnight-blue": #036,
    //   "wafer": #e1d7d2
    // )
    ===
    // _variables.sass
    $hopbush: #c69
    $midnight-blue: #036
    $wafer: #e1d7d2
    ---
    @use "sass:meta"

    @use "variables"

    @debug meta.module-variables("variables")
    // (
    //   "hopbush": #c69,
    //   "midnight-blue": #036,
    //   "wafer": #e1d7d2
    // )
  {% endcodeExample %}
{% endfunction %}

{% function 'meta.type-of($value)', 'type-of($value)', 'returns:unquoted string' %}
  Returns the type of `$value`.

  This can return the following values:

  * [`number`](/documentation/values/numbers)
  * [`string`](/documentation/values/strings)
  * [`color`](/documentation/values/colors)
  * [`list`](/documentation/values/lists)
  * [`map`](/documentation/values/maps)
  * [`calculation`](/documentation/values/calculations)
  * [`bool`](/documentation/values/booleans)
  * [`null`](/documentation/values/null)
  * [`function`](/documentation/values/functions)
  * [`arglist`](/documentation/values/lists#argument-lists)

  New possible values may be added in the future. It may return either `list` or
  `map` for `()`, depending on whether or not it was returned by a [map
  function][].

  [map function]: /documentation/modules/map

  {% codeExample 'type-of', false %}
    @debug meta.type-of(10px); // number
    @debug meta.type-of(10px 20px 30px); // list
    @debug meta.type-of(()); // list
    ===
    @debug meta.type-of(10px)  // number
    @debug meta.type-of(10px 20px 30px)  // list
    @debug meta.type-of(())  // list
  {% endcodeExample %}
{% endfunction %}

{% function 'meta.variable-exists($name)', 'variable-exists($name)', 'returns:boolean' %}
  Returns whether a variable named `$name` (without the `$`) exists in the
  current [scope][].

  [scope]: /documentation/variables#scope

  See also [`meta.global-variable-exists()`](#global-variable-exists).

  {% codeExample 'variable-exists', false %}
    @debug meta.variable-exists("var1"); // false

    $var1: value;
    @debug meta.variable-exists("var1"); // true

    h1 {
      // $var2 is local.
      $var2: value;
      @debug meta.variable-exists("var2"); // true
    }
    ===
    @debug meta.variable-exists("var1")  // false

    $var1: value
    @debug meta.variable-exists("var1")  // true

    h1
      // $var2 is local.
      $var2: value
      @debug meta.variable-exists("var2")  // true
  {% endcodeExample %}
{% endfunction %}
