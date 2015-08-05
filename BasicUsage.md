## A Simple Class ##

```
Animal = $.klass({
  init: function( sound ){
    this.sound = sound;
  },
  say: function( msg ){
    alert( this.sound + ', ' + msg );
  }
})
```

  * **init** called automatically during instantiation.

## Instantiating and Using Class ##

```
fido = new Animal( "Ruff" );

fido.say( "Kibbles and Bits" );
// Ruff, Kibbles and Bits
```

## Subclass With Inheritance ##

```
Dog = $.klass( Animal, {
  init: function( name ){
    this.name = name;
    this.up( arguments, 'Woof Woof' );
  },
  say: function( msg ){
    this.up( arguments, this.name + ': ' + this.noise + msg );
  }
})

fido = new Dog( "Fido" );
fido.bark( 'I want Kibbles and Bits' );
// alert => "Fido: Ruff! I want Kibbles and Bits"
```

  * this.up calls superclass
  * 'arguments' must be passed in. Ugly but lightest way to get code superclass calling to work.

## Using Binding ##