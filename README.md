# Pim's Code Style and Readability Guide

> :information_source: This document is a _living_ document, I will probably add/change/remove things over

## Introduction

This document is my personal view on how you should write code to improve it's readability. Every developer will have their own thoughts / preference and this is mine.

Although the code examples are mostly in javascript, these recommendation should apply to any programming language.

### Why this guide?

Writing this guide is mostly just my way for structuring my thoughts about code readability issues I have experienced.

Good code readability is not just good for other developers, it's also good for yourself as often in two weeks time you will have forgotten all about this one piece of code you wrote. Good code readability also prevents bugs, as the easier it is to read code the easier it is to understand what's going on.

Also when working with other developers, I often find myself explaining the same things when I review code. This document will make it easier for me when reviewing as I can just post a link instead of having to write the same suggestions over and over again.

### Should I read this guide?

That's up to you, but if you do don't use this guide as your only source of truth. The things written down in this guide is what I (currently) believe is the best approach, but it's perfectly ok for you to disagree. Feel free to open an issue and discuss things.

### You write _this_, but this other guide with 5k+ stars says _that_

Yeah, probably a case of _agree to disagree_. Again, feel free to open an issue to discuss

### I checked the code in your _repoX_ and you aren't using your own recommendations

~~One word~~ Two words: _progressive insights_

Also, this is a guide not a rulebook. For me the emphasis is not on _always writing the perfect code_, but more on _knowing when you are not writing perfect code and then being able to make a deliberate decision whether it is needed to refactor your code or to just choose not to_.

### Conflicting recommendations

There will be recommendations that are conflicting with each other. I will try to name them and explain when I like to choose which recommendation.

## Code Structure

### Reading code is easier from top to bottom then from left to right.

This is also the main reason why you always see recommendations about not using more than 80 or 120 chars per line.

That said I don't think that 80/120 chars should ever be enforced and that it's perfectly fine to have longer lines as long as you are adhering to the other recommendations below.

### Do not write condensed code

Condensed code is much harder to read as its more difficult to separate code sections. Try to use line-breaks around code blocks to make it easier on the eye.

If you need to condense code for production, just use a code compressor tool for your programming language.

```js
// BAD
if (true) callFunction()
else callOther()
for(const item of items) item.call()
return true

// GOOD
if (true) {
  callFunction()
} else {
  callOther()
}

for (const item of items) {
  item.call()
}

return true
```

### Write consistent code

> Using a linter / style checker should help you with this!

In a lot of cases it doesn't matter much if you use `A` or `B`, but what is important is that if you use `A` you always use `A` and not a mix of `A` and `B`.

Consistent code structures helps you and other developers to easier/quicker understand the code and lessening the impact of unexpected deviations which can be easily overlooked during reviewing.

### Eliminate ambiguities

It is very important to clearly indicate your intentions in your code. Either by descriptive variable naming, using the proper code syntax or by just adding a comment. This is also related to writing consistent code.

```js
// BAD
if (user.name.test(/^[0-9]/)) { // why do we need to return false when the name starts with a gigi
  return false
}

if (user.name == '') { // but what when name is the integer 0 and not a string?
if (user.name === '') { // but what about when user.name is null?

// GOOD
const isMachineUser = user.name.test(/^[0-9]/)
if (isMachineUser) {
  return false
}

if (user.name) {

// BETTER
const isMachineUser = user.name.test(/^[0-9]/)
if (isMachineUser) {
  // Only machine users have a name starting with a digit,
  // this is enforced by a validation rule when users sign-up
  // return false because the upcoming logic in this function
  // doesn't apply to them
  return false
}
```

### Keep the main code path on the main code branch

Each function you write should have a single responsibility. Make sure that the successful execution of this responsibility is always on the main code branch, and only branch out in your code for conditional control structures, error handling etc.

```js
// BAD
function saveUser(user) {
  if (validateUser(user)) {
    const result = api.post(user)
    if (result) {
      return true
    } else {
      return false
    }
  } else {
    return false
  }
}

// GOOD
function saveUser(user) {
  if (!validateUser(user)) {
    return false
  }

  const result = api.post(user)
  if (!result) {
    return false
  }

  return true
}
```

### Prevent nesting multiple control flow statements

When writing functions, try to prevent nesting control flow statements. I.e. try to have a maximum of two additional indentation levels, but prefer one especially when it comes to `IF`s

Use the concept of _early returns_ to manage this.

> Within a loop I consider using `continue` or `break` also as an _early return_

```js
// BAD
function doSomething() {
  if (true) {
    if (false) {
      if (true) {
        return true
      } else {
        return false
      }
    } else {
      return false
    }
  } else {
    return false
  }
}

foreach (const item of items) {
  if (item.value) {
    processItem(item)
  }
}

// GOOD
function doSomething() {
  if (!true) {
    return false
  }

  if (!false) {
    return false
  }

  if (!true) {
    return false
  }

  return true
}

foreach (const item of items) {
  if (!item.value) {
    continue
  }

  processItem(item)
}
```

### One logic statement per line

This applies both to code as definitions. When possible, also make sure to use a logical order.

There is no government tax on the number of lines of code you write (at least not yet), so use those lines.

This approach is also better when checking for line coverage. Eg if in the example below you would throw an exception in `callMethod1` during tests (which is catched by a parent), you will never know that `callMethod2` is never tested.

```js
// BAD
const myArray = ['abc', 'xyz', 'def'] // no logical order
const myObject.callMethod1().callMethod2()
const isInRange = value => (value > 0 && value < 100)

// GOOD
const myArray = [ // also in logical (alphabetical) order!
  'abc',
  'def',
  'xyz'
]

const myObject
  .callMethod1()
  .callMethod2()

function inRange(value) {
  if (value < 0) {
    return false
  }

  if (value > 100) {
    return false
  }

  return true
}
```

## `IFs`

### Use `truthy` conditions

```js
// BAD
if (!false) {
if (user.name.length > 0) {
if (user !== null) {

// GOOD
if (true) {
if (user.name) {
if (user) {
```
> *Possibly conflicting with `Eliminate ambiguities`*
> You could argue that `user.name.length > 0` is less ambiguous then just `user.name`, and you are probably right. But using `.length` means that you need to absolute sure that `user.name` is also a _string_, because otherwise you will get an error.
> Therefore this recommendation has often preference above `Eliminate ambiguities`


> *Possibly conflicting with `Keep the main code path on the main code branch`*
> The code examples already clearly show why `Keep the main code path on the main code branch` should have preference over the recommendation of using truthy conditions.

### Use type strict comparisons

Always use type script comparisons in loosely typed languages like JS and PHP. This both prevents unnecessary type conversions as it prevents a lot of ambiguity about the intentions of your code

### Do not use ternaries (`inline if`'s)

Again, there is no government tax on the number of lines of code you write. So don't be lazy and just write out the if statement. Its much better readable

Maybe even more importantly, this also greatly improves checking for code coverage. In my experience in real life projects the focus is mostly on line-coverage and often it's unlikely that you will get 100% branch coverage. Especially when you are often using multiple conditions in your if's.
This means that you will be expecting your branch coverage to be less than 100%. When you then use an inline-if on e.g. a return statement, you are effectively preventing yourself from detecting whether you have tested all possible return statements (as if you'd only test one return value, the line coverage will already be ok).

```js
// BAD
return val >= 0 && val <= 100

// GOOD
if (val < 0) {
  return false
}

if (val > 100) {
  return false
}

return true
```

### Dont use long/multiple conditions
If you need to use long or multiple conditions, create a boolean variable first with a descriptive name. Then use that variable for the `if`.

```js
// BAD
if (!notDeleted && (object.key.name === 'test' || other-object.key.name === 'test') {
  callSomeFunction()
}

// GOOD
const shouldCallFunction = !notDeleted && (object.key.name === 'test' || other-object.key.name === 'test'
if (shouldCallFunction) {
  callSomeFunction()
}
```
### Avoid using `else`

In 95%+ of the cases using an `else` shouldnt be necessary, most often you can refactor your code to use an _early-return_ instead (see `Prevent nesting multiple control flow statements`)

### Don't use `else-if`

In 98%+ of the cases using an `else-if` isn't necessary, for code readability it would be better to split `else-if`'s in separate `if`'s with early returns.
If you are using `else-if` regurlarly then that's often an indication that you just need to refactor your code into smaller functions
