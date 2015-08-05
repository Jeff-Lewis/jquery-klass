# Sunny Hirai's Module Pattern #

The purpose of this article is to show you a useful variation on the JavaScript module pattern introduced by Douglas Crockford.

While the module pattern works well, it can be modified to increase code quality, improve readability and reduce some types of bugs.

Some problems are:

  * Ambiguity in where to place 'constructor' code (differs depending on the constructor's role)
  * Different ways to call public and private members means you have to remember the member's scope before referring to it.
  * Since public/private members are declared differently, it takes time to convert a member from public to private and visa versa. This is exasperated by the fact that you don't always know ahead of time whether a member should be public or private.
  * The pattern is not self documenting and prefers commenting
  * The pattern is a little ugly (though some like its raggedness)

### Animal Module ###

To demonstrate the regular Module Pattern and my modified version, I've created a simple animal class. It has the following members:

  * 'name' which is private
  * 'output' a private method to output text to the user
  * 'say' a public method
  * 'howl' a public method that calls 'say'

### Module Pattern ###

```
  Animal = function( name ){

    // private variables
    var name = name;

    // private functions
    function output( msg ){
      alert( msg );
    }

    // public members
    return {
      say: function( msg ){
        output( name + ': ' + msg );
      },
      howl: function( msg ){
        returnObject.say( 'awoooo... ' + msg );
      }
    };
  }
```

### Sunny Hirai's Module Pattern ###

```

  Animal = function( name ){
    var me = {
      init: function(){
        me.name = name;
      },
      output: function( msg ){
        alert( msg );
      },
      say: function( msg ){
        me.output( me.name + ': ' + msg );
      },
      howl: function( msg ){
        me.say( 'awoooo... ' + msg );
      }
    }
    me.init();
    return { say: me.say, howl: me.howl }; // public methods
  }

  // NOTE: An alternate for "return {}" code below for about 200 bytes of code.
  // jQuery users could add it as a $.publish plugin. See bottom of page for code.
  //
  // return publish( me, 'say', 'howl' );
  //
  // or
  //
  // return $publish( me, 'say', 'howl' );

```

## Differences ##

Now that you've seen the code, here are the benefits of the new module pattern.

  1. **Neat:** It's nice and neat and easier to read. It's easy to see what all the methods are because they are all together and nicely formatted. The module pattern is a little more efficient (less lines of code) but the private functions are declared quite differently from the public ones.

### Constructors Without Thinking ###

Where should the constructor code go in the module pattern? Is it before the public object declaration, after or both before and after?

The answer is it depends on what it does. By convention, it's nice to have the constructor code at the top since it is the first thing that is done and therefore having it at the top is intuitive; however, if the constructor calls a public method in the public object (e.g. 'say' or 'howl') then the constructor needs to go below. This is because the public methods do not exist until after they are declared.

Here are the code changes needed in the original Module pattern:

```
  Animal = function( name ){

    // private variables
    var name = name;

    // private functions
    function output( msg ){
      alert( msg );
    }

    // public members
-    return {
+    var pub = {
      say: function( msg ){
        output( name + ': ' + msg );
      },
      howl: function( msg ){
        returnObject.say( 'awoooo... ' + msg );
      }
    };

+   // constructor
+   pub.say( name + ' is born' );

+   return pub;

  }
```

Note also that the 'output' private function is above the public members. It can be moved below the public members but must stay above the constructor. This bevy of rules is what adds to the confusion.

This isn't a big deal when the module is small (as ours is), but as the complexity of the module grows, it becomes more difficult to keep track of the order of constructors and declarations.

The variation on the module pattern sidesteps all of this by defining all the members (both public and private) first including the constructor (which I've named 'init') and then executes 'me.init()' at the very end. It is easy to read and intuitive to understand.

### Last Minute Private Variables ###

There are a few benefits (and one gotcha) to creating a sub-object and using that as the public object.

The first benefit is that during development, you can return the entire object and therefore every member is public. This is useful during development because it makes it easy to inspect your object. When you are ready to release the code as production code, you then choose which methods should be kept private. In fact, I find this more harmonious with the way I work. I want to get it working first and then figure out what I need to expose to the public.

In our Animal example, let's say we wanted to make the 'say' method private. Here's the change in the new Module Pattern.

```
- return { say: me.say, howl: me.howl };
+ return { howl: me.howl };
```

or with the limitKeys() method available:

```
- return limitKeys( me, 'say', 'howl' );
+ return limitKeys( me, 'howl' );
```


In the original module pattern, this is the change.

```
- say: function( msg ){
-   output( name + ': ' + msg );
- },
+ function say( msg ){
+   output( name + ': ' + msg );
+ }
// Also in the howl method
- me.say( 'awoooo... ' + msg );
+ say( 'awoooo... ' + msg );
```

When you make the change in the module pattern, you need to (a) rewrite the method to fit into the new way of declaring the method in the scope and (b) rewrite all the method calls.

As the size of your module grows, the complexity increases.

### Gotcha of Last Minute Private Variables ###

One gotcha of returning a different object as a public object is that changing values in the public object won't actually change the 'me' values. For example, you might get a copy of the 'name' variable in the public object, but if you change the 'name' variable's value, it won't change in the 'me' object.

```
  // the me object might look like this:
  {
    name: 'fido',
    output: function(){...},
    say: function(){...},
    howl: function(){...}
  }
  // the public object might look like this if you exposed 'name'
  {
    name: 'fido',
    say: function(){...},
    howl: function(){...}
  }
  // if you change the public object's name, you WON'T change the
  // 'me' object's name.
```

The solution is to never expose variables only methods in the public object. Although this is a gotcha, it encourages better Object Oriented Programming. All variables can still be accessed by getters and setters.

In the short-run, I sometimes find this annoying but in the long run, getters and setters saves me. I've found I've just started always using them, even before I created this pattern. In fact, I discovered this gotcha while writing, not during usage.

### Better Iterative Development ###

## Comparison ##

The 'publish' code for JavaScript and for jQuery:

```
// Pure JavaScript version
function publish( source ){
  var dest = {}, key, member;
  for (var i=1;i<arguments.length;i++){
    key = arguments[i];
    member = source[ key ];
    if (member instanceof Function) throw "You cannot publish non-functions.";
    dest[ key ] = member;
  }
  return dest;
}

// jQuery version
$.extend({
  publish: function( source ){
    var dest = {}, key, member;
    for (var i=1;i<arguments.length;i++){
      key = arguments[i];
      member = source[ key ];
      if (member instanceof Function) throw "You cannot publish non-functions.";
      dest[ key ] = member;
    }
    return dest;
  }
})
```