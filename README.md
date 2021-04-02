# rust-cheatsheet

This has been sitting on my laptop for ages. It's a collation of all the rust syntax that confused me when I first read the rust book. Hope people find it useful!


---

```rust
struct StructName<T> {
  ...
}

impl<T> StructName<T> {
  ...
  // implement methods/assoc_funcs for this struct which are generic over all types.
}

impl<T: Trait1 + Trait2> StructName<T> {
  ...
  // implement methods/assoc_funcs for this struct which are generic ONLY over types which implement both traits 1 and 2.
}

// conditionally implementing a trait:
impl<T: IfItAlreadyImplementsThisTrait> AlsoImplementThisTrait for T {}
```

---

```rust
trait TraitName {
  fn method_name(&self) -> SomeTypeName; // we could include a default function implementation body here, as well.

  fn assoc_func_name() -> Self; // usually returns an instance of the type. we could include a default function implementation body here, as well.
}

impl TraitName for TypeName {
  fn method_name(&self) -> SomeTypeName {
    ...
  }

  fn assoc_func_name() -> Self {
    ...
  }
}

impl TraitName for OtherTypeName {} // empy block means "use the default" (as specified in the trait definition)
```


--


```rust
trait TraitName<T=DefaultType> {
  ...
}

struct TypeName1
struct TypeName2

impl TraitName for TypeName1 {
  // defaults to using the DefaultType
}

impl TraitName<ConcreteType> for TypeName2 {
  // uses the specified type
}
```


---


Putting a lot of it together:

```rust
struct StructName<T> {
  ...
}

trait TraitName<U=DefaultType> {
  ...
}

impl<T> TraitName<ConcreteType> for StructName<T> {
  ...
  // implement methods/assoc_funcs for this struct which are generic over all types.
  // methods/assoc_funcs might refer to the specified ConcreteType (rather than the DefaultType) too
}

impl<T: Trait1 + Trait2> TraitName<ConcreteType> for StructName<T> {
  ...
  // implement methods/assoc_funcs for this struct which are generic ONLY over types which implement both traits 1 and 2.
  // methods/assoc_funcs might refer to the specified ConcreteType (rather than the DefaultType) too
}

// conditionally implementing a trait:
impl<T: IfItAlreadyImplementsThisTrait> AlsoImplementThisTrait<ConcreteType> for T {}
```


---


```rust
// grouped function signatures achieve the same thing:

fn func_name(param: impl TraitName) {}
fn func_name<T: TraitName>(param: T) {}

fn func_name(param1: impl Trait1, param2: impl Trait2) {}

fn func_name<T: TraitName>(param1: T, param2: T) {}

fn func_name(param: impl Trait1 + Trait2) {}
fn func_name<T: Trait1 + Trait2>(param: T) {}
fn func_name<T>(param: T) -> ReturnType
  where T: Trait1 + Trait2
{
...
}

fn func_name<T: Trait1 + Trait2, U: Trait3 + Trait4>(t: T, u: U) {}
fn func_name<T, U>(t: T, u: U) -> ReturnType
  where T: Trait1 + Trait2
        U: Trait3 + Trait4
{
...
}

// return a Type that implements a particular Trait:
fn func_name() -> impl TraitName {}
```

---

```rust
trait TraitName {
  type AssocTypeName; // associated type placeholder name

  // add function signatures of methods and assoc functions which refer to AssocTypeName
}

impl TraitName for SomeTypeName {
  type AssocTypeName = ConcreteType; // e.g. u32

  // add implementations of methods and functions which work with AssocTypeName as u32 (or whichever ConcreteType is specified)
}
```


---

```rust
trait TraitName {
  type AssocTypeName: Trait1; // associated type placeholder name, where the type must implement Trait1

  // add function signatures of methods and assoc functions which refer to AssocTypeName
}

impl TraitName for SomeTypeName {
  type AssocTypeName = ConcreteType; // where the ConcreteType must implement Trait1

  // add implementations of methods and functions which work with AssocTypeName as ConcreteType
}
```


---


```rust
trait TraitName<T> {
  // add function signatures of methods and assoc functions which refer to T
}

impl<ConcreteType1> TraitName<ConcreteType1> for SomeType {
  // add implementations of methods and functions which work with ConcreteType1
}

impl<ConcreteType2> TraitName<ConcreteType2> for SomeType {
  // add implementations of methods and functions which work with ConcreteType2
}
```

___


```rust
trait TraitName: SuperTraitName {
  ...
}

// ^^ each time we implement this trait for some type, we must also implement the supertrait for that type too! This trait's methods _depend_ on the supertrait in some way.
```


---


Calling a method or associated function on an instance of a type:

```rust
let instance: TypeName = ...
instance.method_name(); // calls a method

TraitName::method_name(&instance); // also calls a method in the same way. Disambiguates if multiple traits have the same method name.

<TypeName as TraitName>::method_name(instance, arg1, ...); // calls a method. Disambiguates when multiple types have traits which include this method name. I.e. when multiple impl blocks have the same function name.

<TypeName as TraitName>::assoc_func_name(arg1, ...); // calls an associated function. Disambiguates same as above.
```

---

```
use path::to::ExternalTraitName;
```

---

`for<'a>` can be read as "for all choices of 'a" - it is an example of a Higher-Ranked Trait Bound (HRTB) - and basically produces an infinite list of trait bounds that must be satisfied
