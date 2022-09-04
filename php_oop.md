# php

object oriented php notes

- ```php
  class Fruit{
      public $name;
      public $color;
      ...
  }
  ```

- must prefix the variables with access modifiers

  - `public`: property or method can be accessed from everywhere
  - `protected`: can only be accessed **within the class** and subclasses
  - `private`: can only be accessed within the class

- ```php
  function set_name($name){
      $this->name = $name;
  }
  
  function get_name(){
      return $this->name;
  }
  
  ...
  
  // making object and accessing functions
  $apple = new Fruit();
  $apple->set_name('Apple');
  // accessing variables
  $apple->name = 'Apple';
  ```

- `$this` refers to current object and available inside the methods

- `instanceof` to check if object belongs to specific class
  `var_dump($apple instanceof Fruit)`

-  ```php
   function __construct($name, $color){
       $this->name = $name;
       $this->color = $color;
   }
   ```

- ```php
  // destructor function is automatically called when the script stops or exits
  function __destruct(){
      echo "This fruit is {$this->name}";
  }
  ```

- to call internal methods: `$this->function();`

- `static` properties and methods do not need an instance of the class to be called
  to call `static` methods: 
  external class - `ClassName::staticMethod();` 
  internally - `self::staticMethod();`
  parent - `parent::staticMethod();`
  
- `::` for class-level properties
  `->` for object-level properties

## inheritance

- `extends`

- remember that `protected` modifier means **within the class** and its subclasses

- To prevent class inheritance or method overriding, use the `final` keyword
  `final class ClassName`
  `final public function functionName()`

- ```php
  abstract class ParentClass{
      abstract public function ..... : return type;
      abstract protected function prefixName($name);
  }
  
  //child class can define optional arguments that are not in parent's abstract methods
  class ChildClass{
      public function prefixName($name, $separator=".",$greet="Dear"){
  		...
          return "{$greet}{$prefix}{$separator}{$name}";
      }
  }
  ```

- Notes on abstract classes:

  - child classes must have same or less restricted access modifiers

- `implements` for interfaces (can implement multiple interfaces)

- PHP only support **single inheritance** (extending one class)
  Use **traits** if class needs to inherit multiple behaviors

  ```php
  trait TraitName{
      ....
  }
  
  trait TraitName2{
      ....
  }
  
  class Class{
      use TraitName, TraitName2;
  }
  ```

## constants

- case-sensitive, capitalized convention

- ```php
  class Hello{
      const MESSAGE = "Hello!";
  }
  
  echo Hello::MESSAGE;
  ```

- `::` - scope resolution operator 
  Access constant from outside the class
  If inside the class:
  `echo self::MESSAGE;`

## namespaces

1. allow same name to be used for more than one class
2. all functions and classes now belong to the namespace

```php
namespace Namespace;

// nested namespaces (declaring namespace Html inside namespace Code)
namespace Code\Html;

// to access classes from different namespace 
$table = new Html\Table();
//or
namespace Html; // declare namespace within namespace
$table = new Table();

use Html as H; // alias
$table = new H\Table();
use Html\Table as T; // give alias to a class
$table = new T();
```

