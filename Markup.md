# Caffeine - Go Markup Bullets #

Approaches to parsing nested markup like bullets, numbered lists and indented code. Must handle the following.

```
* bullet
* same level
  * optional spaces okay
* hanging lines
  are okay
** indent
**** multi-level indent
*** outdent
* multi-level outdent
```

## Nested Rule Extensions ##

  1. name: ul
  1. regex: `(?<=\n|\A)(\s*[*]+\s.*?)\n\s*\n+`
  1. template: $1
  1. item\_regex: `match start with * + any lines that don't start with *`
  1. item\_template: e.g. `<li>$1</li>`
  1. nest\_regex: `match start with * but do not include the first * in match`
  1. nest\_template: e.g. `<ul>$1</ul>`

Pros

  * Easy to write grammar
  * No need to add new grammar

Cons

  * Does not support UL and OL mixed bullets

## Nested Grammar (though experiment) ##

We need to make a grammar that has a multiple rules to differentiate OL and UL lists.

  1. name: ul
  1. regex: `(?<=\n|\A)(\s*[*]+\s.*?)\n\s*\n+`
  1. template: $1
  1. item\_regex: `match start with * + any lines that don't start with *`
  1. item\_template: e.g. `<li>$1</li>`
  1. nest\_regex: `match start with * but do not include the first * in match`
  1. nest\_template: e.g. `<ul>$1</ul>`

Pros

  * Easy to write grammar
  * No need to add new grammar

Cons

  * Does not support UL and OL mixed bullets




## Reparse ##

  1. Use a regexp to capture and parse one indent level at a time.
  1. The problem is that this requires the date in a multiple match. e.g. `(<=\n)([*] (.*))*`
  1. We want the second match to return ALL of the sub-matches but it only returns the last one
  1. The only way around this that I can think of is to parse the regex and handle it programmatically. That is, take the regex, find the `(regex)*` part and turn it into multiple match statements.

Pros

  * Only one grammar and no new rules
  * Supports mixed bullets UL and OL

Cons

  * Hard to implement
  * Possibly hard to write the grammar (not intuitive)