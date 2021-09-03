# OCaml Notes

----------------

## Table of contents

<!-- TOC depthFrom:0 depthTo:6 withLinks:1 updateOnSave:1 orderedList:0 -->

- [OCaml Notes](#ocaml-notes)
  - [Table of contents](#table-of-contents)
  - [Forward](#forward)
  - [Compiling](#compiling)
    - [`ocaml`](#ocaml)
    - [`ocamlopt`](#ocamlopt)
    - [`dune`](#dune)
  - [Basic program structure](#basic-program-structure)
    - [Syntax](#syntax)
      - [Statements](#statements)
      - [Expressions](#expressions)
    - [Semicolons and `in`](#semicolons-and-in)
    - [End of input delimiter `;;`](#end-of-input-delimiter-)
    - [Parenthesis](#parenthesis)
    - [Nesting](#nesting)
    - [Recursion](#recursion)
    - [Mutually recursive bindings](#mutually-recursive-bindings)
    - [Scope](#scope)
    - [Empty return values](#empty-return-values)
  - [Functions](#functions)
    - [Currying](#currying)
    - [Labeled arguments](#labeled-arguments)
    - [Composition operators (`@@` and `|>`)](#composition-operators--and-)
    - [Function vs fun](#function-vs-fun)
  - [Tuples](#tuples)
    - [Lists](#lists)
  - [Arrays](#arrays)
    - [Initialization](#initialization)
    - [Get](#get)
    - [Set](#set)
    - [Iterate](#iterate)
    - [Fold](#fold)
  - [Maps](#maps)
  - [Sets](#sets)
  - [Type definitions](#type-definitions)
    - [Product type](#product-type)
    - [Sum type](#sum-type)
    - [Algebraic types](#algebraic-types)
  - [Parametric polymorphism](#parametric-polymorphism)
  - [Records](#records)
  - [Monadic/applicative types](#monadicapplicative-types)
    - [Map](#map)
    - [% syntax](#-syntax)
    - [Bind](#bind)
  - [Modules](#modules)
  - [Submodules](#submodules)
  - [Interfaces](#interfaces)
    - [Signatures](#signatures)
    - [Abstract types](#abstract-types)
      - [Encapsulation](#encapsulation)
      - [Preventing invariant violations](#preventing-invariant-violations)
  - [utop](#utop)
    - [printing](#printing)
    - [UInt32](#uint32)
  - [Useful modules](#useful-modules)
  - [Libraries](#libraries)
    - [Show](#show)

<!-- /TOC -->

## Forward

These notes were compiled from various sources with the aim of being a learning aid and quick reference.  The descriptions and explanations are a work in progress.

## Compiling

There are many ways to compile and run OCaml programs.

| Command  | Use |
|----------|-----|
| `ocaml`    | Non-interactive interpreter |
| `ocamlopt` | Generate machine code |
| `ocamlc`   | Generate bytecode and interfaces |
| `utop`     | Interactively experiment with code |
| `dune`     | Build larger projects with many dependencies |

### `ocaml`

Example of using the non-interactive interpreter

*hello_world.ml*

```ocaml
print_endline "Hello World!"
```

*Run*

```text
ocaml hello_world.ml
```

*Output*

```text
Hello World!
```

### `ocamlopt`

Example compile command

*Compile*

```text
ocamlopt -o hello_world hello_world.ml
```

*Run*

```text
./hello_world
```

*Output*

```text
Hello World!
```

### `dune`

Dune builds a project by reading a `dune` file, similar to a Makefile.

For example, assuming a simple project consisting of `main.ml` and using a single library `ppx_deriving.show`, the `dune` file might look like this.

```text
(executable
 (name main)
 (libraries core)
 (preprocess
  (pps ppx_deriving.show)))
```

The build command would be

```text
dune build ./main.exe
```

To build and run the code you could use this command.

```text
dune exec ./main.exe
```

## Basic program structure

This section tries to explain the nature of OCaml program structure and common misconceptions.

At a high level, OCaml is an functional language and, in contrast to imperative languages, does not concern itself very much with the order that things happen.  In many ways OCaml program just describes the functions that need to be applied to obtain the desired behaviour.

### Syntax

When writing software in OCaml you think of your program as a sequence of function calls.

```text
f1()
f2()
...
fn()
```

A function call must have a *return value* and may have additional *side-effects*.

**Return values**
A return value is something returned by a function, for example, and integer or a tuple.

**Side-effects**
A side-effect is something a function does besides returning a value, such as printing a value, modifying a reference or creating a file.

**Representing nothing**
We say that a function's return value is *uninteresting* when it is nothing or empty (e.g. `NULL` in C parlance); however, as learnt above, functions must always have a return value!

In this case, the function must return a value of type `unit`.  There is only one value of the type `unit` and that value is `()` -- you can think it as the "nothing" value.

#### Statements

OCaml does not have statements as you would find in an imperative language.  Nevertheless, the funtionality and behaviour of statements can be provided with expressions whose values are uninteresting (i.e. `unit` type).

#### Expressions

The most important expression is called a *let definition*, which is used to give names to values and functions.  Since it is used to give names to functions and an OCaml program is nothing more than a collection of functions, the let syntax is the anchor point of everything in OCaml programs.

```ocaml
let there_be_code =
  do something useful in
  a functional way
```

There are other expressions for flow of control, pattern matching, exception handling and defining types, just to name a few, but in this section we will focus exclusively on let expressions.

`let`

The syntax is

```ocaml
let name = expr1 in expr2 in ... exprN
```

### Semicolons and `in`

Semicolons is an expression separator, not a statement terminator!

This code

```ocaml
let inc x =
    print_endline "Incrementing is fun";
    print_endline "when you just add one!";
    x + 1
```

is semantically equivalent to

```ocaml
let inc x =
    let () = print_endline "Incrementing is fun" in
    let () = print_endline "when you just add one!" in
    x + 1
```

### End of input delimiter `;;`

The end of input delimiter `;;` it is not part of the OCaml language and is only used by the interpreter.  It is a toplevel token that means "end of declaration" and cannot be put in the middle of an expression.  It is a common mistake to confuse it for end of statement, but should rather be thought as a "get back to global scope" instruction.

Note that if you want to use ocaml in non-interactive interpreter mode (i.e. `ocaml hello_world.ml`) then you need to use `;;` after directives, such as `#use "topfind";;` -- directives are not part of the language either!

### Parenthesis

Expressions can be wrapped in parenthesis or *being* and *end* keywords.  For example,

```ocaml
let sum_it n =
  n*(n + 1)/2
let _ = Printf.printf "%d\n" (sum_it 10)
```

is semantically equivalent to

```ocaml
let sum_it n =
  n * begin n + 1 end / 2
let _ = Printf.printf "%d\n" (sum_it 10)
```

### Nesting

Let expressions can be nested like so

```ocaml
let foo = 3 in
  let bar = 3 in
  foo * bar
```

Nested items have limited scope.  The general form is

`let` `variable` `in` `{` `scope` `}`

### Recursion

Sometimes it is helpful to define a let binding recursively.  The syntax is

```ocaml
let rec sum n =
  if n == 1 then
    1
  else
    n + (sum (n - 1))
```

### Mutually recursive bindings

The `let ... and ...` syntax is used for mutually recursive bindings.

Why do we need this?  Mutually recursive function definitions like this

```ocaml
let rec g x =
  match x with
    | 0 -> ""
    | _ -> String.concat "" ["GNU's "; nu (x - 1)]
let nu x =
  match x with
    | 0 -> ""
    | _ -> String.concat " " ["Not Unix!"; g (x - 1)]
let () = Printf.printf "%s" (g 10)
```

will fail with `Error: Unbound value nu`.

If instead we use `let ... and ...` syntax we are all good.

```ocaml
let rec g x =
  match x with
    | 0 -> ""
    | _ -> String.concat "" ["GNU's "; nu (x - 1)]
and nu x =
  match x with
    | 0 -> ""
    | _ -> String.concat " " ["Not Unix!"; g (x - 1)]
let () = Printf.printf "%s" (g 10)
```

Output

```text
GNU's Not Unix! GNU's Not Unix! GNU's Not Unix! GNU's Not Unix! GNU's Not Unix!
```

### Scope

Every let binding opens a new scope.  Top-level bindings (i.e. those at the top-level scope of a file) continue until the end of the file.

Shadowed declarations do not impact bindings outside of the current scope.

For example,

```ocaml
let x = 3
let foo() = Printf.printf "%d\n" x

let x = x * 3
let bar() = Printf.printf "%d\n" x

let _ = foo()
let _ = bar()
```

outputs

```text
3
9
```

### Empty return values

If you want to execute something and don't care about its return value, you should wrap it with a let-expression and match it against the wildcard pattern like this

```ocaml
let _ = print_endline "Hello World!"
let _ = exit 0
```

If you call a function that returns unit you can also match against the unit value `()`, for example

```ocaml
let () = print_endline "Hello World!"
```

The advantage over the wildcard is that if you accidentally write an expression that has type other than unit you will get a type error from the compiler.

## Functions

```ocaml
let ratio (x : int) (y : int) : float = Float.(of_int x / of_int y);;
val ratio : int -> int -> float = <fun>
```

```ocaml
let ratio f (x : int) (y : int) : float = Float.(of_int (f x) / of_int (f y));;
val ratio : (int -> int) -> int -> int -> float = <fun>
```

### Currying

Partial application of functions is very easy in OCaml.  Just omit some arguments and what you obtain is a new function.

For example,

```ocaml
let add a b = a + b
```

is symatically equivalent to

```ocaml
let add =
  fun a ->
    fun b -> a + b
```

Thus, function currying is as simple as

```ocaml
let increment = add 1;;
```

Here `increment` is actually a *closure*, which is a function and an environment where name `a` is bound to value `1`

### Labeled arguments

Labeling function arguments and explicitly stating their types aids code clarity.

```ocaml
let foo ~x:(x : int) ~y:(y : int) : int =
  x + y
;;
val foo : x:int -> y:int -> int = <fun>
```

```ocaml
foo ~y:3 ~x:2;;
- : int = 5
```

### Composition operators (`@@` and `|>`)

The syntax

- `f @@ g @@ x`
- `x |> g |> f`

are both equivalent to

`f(g(x))`

### Function vs fun

Similarly, `function` is to `fun` what `match` is to `let`.

The `function` syntax allows multiple branches to be defined at the function level.

So

```ocaml
function
| A (x, y) -> x + y
| B x -> x
| C -> 0
```

is equivalent to

```ocaml
fun v ->
match v with
| A (x, y) -> x + y
| B x -> x
| C -> 0
```

## Tuples

Tuples store fixed numbers of items of different types.  The items are separated by commas.

Commas create tuples even if there's no parenthesis around them, but this is not good practise.

```ocaml
let t = (3, "four")
val t : int * string = (3, "four")
```

```ocaml
(* Poor style and should be avoided *)
let t = 3, "four"
val t : int * string = (3, "four")
```

The `*` in the type specifier notation is the Cartesian product.

For example, the set of all pairs containing one int and one string.

```ocaml
let (i,s) = t;;
val i : int = 3
val s : string = "four"
```

```ocaml
String.length s
- : int = 4
```

```ocaml
let distance ((x1 : float), (x2 : float)) ((y1 : float), (y2 : float)) : float =
   Float.(sqrt ((x1 - x2)**2.0 + (y1 - y2)**2.0))
;;
```

```ocaml
let distance ((x1 : float), (x2 : float)) ((y1 : float), (y2 : float)) : float =
   Float.sqrt((x1 -. x2)**.2. +. (y1 -. y2)**.2.)
;;
```

## Lists

Lists let you hold any number of items of the same type.  The items are separated by semicolons.

### Initialization

```ocaml
let languages = ["C++"; "C"; "Rust"; "OCaml"]
val languages : string list = ["C++"; "C"; "Rust"; "OCaml"]
```

```ocaml
List.map languages ~f:String.length
- : int list = [3; 1; 4; 5]
```

List constructor

```ocaml
let foo = ["English"];;
val foo : string list = ["English"]
```

```ocaml
"French" :: "Spanish" :: foo;;
- : string list = ["French"; "Spanish"; "English"]
```

Or

```ocaml
(* Create a list of fives nines *)
let l = List.init 5 ~f:(fun _ -> 9) ;;
```

Bracket notation is just syntactic sugar

```ocaml
1 :: (2 :: (3 :: []));;
- : int list = [1; 2; 3]
```

```ocaml
1 :: 2 :: 3 :: [];;
- : int list = [1; 2; 3]
```

### Pattern matching

```ocaml
let my_fav langs =
  match langs with
    | first :: the_rest -> first
    | [] -> "OCaml"
;;
val my_fav : string list -> string = <fun>
```

### Recursive list functions

```ocaml
let rec sum l =
  match l with
    | [] -> 0
    | hd :: tl -> hd + sum tl
;;
val sum : int list -> int = <fun>
```

Or better with explicit types

```ocaml
let rec sum (l : int list) : int =
  match l with
    | [] -> 0
    | hd :: tl -> hd + sum tl
;;
val sum : int list -> int = <fun>
```

```ocaml
let l = [1;2;3;4;5;6;7;8;9;10];;
val l : int list = [1; 2; 3; 4; 5; 6; 7; 8; 9; 10]
sum l;;
- : int = 55
```

## Arrays

Arrays contain a fixed number of items of the same type.

### Initialization

```ocaml
let a = [| 3; 1; 4; 5; 9 |]
```
or
```ocaml
(* Create an array of ten threes *)
let a = Array.init 10 ~f:(fun _ -> 3) ;;
```

### Get

Element access works like this

```ocaml
let a = [| 3; 1; 4; 5; 9 |] in
let () = print_int a.(0)
```

### Set

Update an element like this

```ocaml
let a = [| 3; 1; 4; 5; 9 |] in
a.(0) <- 9
```

### Iterate

```ocaml
let a = [| 3; 1; 4; 5; 9 |] in
Array.iter ~f:(Printf.printf "%d\n") a;;
```

### Fold

```ocmal
let a = [| 3; 1; 4; 5; 9 |] in
Array.fold ~init:0 ~f:(fun sum x -> x + sum) a;;
```

## Maps

Maps store key value pairs.

**Example**

```ocaml
utop # let m = Map.empty (module Int);;
utop # let m = Map.set m ~key:0 ~data:0;;
utop # let m = Map.set m ~key:1 ~data:1;;
utop # let m = Map.set m ~key:2 ~data:2;;
utop # Map.iter m ~f:(fun x -> printf "%d\n" x);;
0
1
2
utop # let m = Map.map m ~f:(fun x -> x + 1000);;
utop # Map.iter m ~f:(fun x -> printf "%d\n" x);;
1000
1001
1002
```

**Another example**

```ocaml
utop # let m = ref (Map.empty (module Int)) in
for i = 0 to 4 do
  m := Map.set !m ~key:i ~data:i
done;
Map.iter !m ~f:(fun x -> printf "%d\n" x);;
0
1
2
3
```

**And another Example**

```ocaml
let m =
  let rec go i map =
    if i < 4 then
      go (i+1) (Map.set map ~key:i ~data:i)
    else
      map in go 0 Int.Map.empty
```

**Finally an absurd example**

```ocaml
utop # let m = Fn.apply_n_times ~n:4 (fun x -> ref (Map.set !x ~key:(Map.length !x) ~data:(Map.length !x))) (ref (Map.empty (module Int))) in
Map.iter !m ~f:(fun x -> printf "%d\n" x);;
0
1
2
3
```

## Sets

Sets store unique items in an unordered fashion.

**Example**

```ocaml
utop # let s = Set.of_list (module Int) [1;2;3];;
val s : (int, Int.comparator_witness) Set.t = <abstr>

utop # Set.to_list s;;
- : int list = [1; 2; 3]

utop # let u = Set.union
(Set.of_list (module Int) [1;2;3])
(Set.of_list (module Int) [3;4;5]);;
val u : (int, Int.comparator_witness) Set.t = <abstr>

utop # Set.to_list u;;
- : int list = [1; 2; 3; 4; 5]
```

## Type definitions

OCaml has some types that may not be familiar to those familiar with imperative languages.

### Product type
In ML the term for a tuple is a *product type*.  OCaml also uses product types to refer to tuples.  For example, consider the definition of a vector

```ocaml
let vec = (0,1,2)
```

 OCaml will refer to this type as

```ocaml
int * int * int
```

### Sum type

This is similar to a variant type.

```ocaml
type transaction = Transfer | Coinbase
```

### Algebraic types

Product and sum types are called *algebraic data types*.

## Parametric polymorphism

Types that are polymorphic have type variables that can be substituted for any type.  Type variables are prefixed with an apostrophe like this `'a`.

Type names are referred to as *type constructors*

```ocaml
type ('a, 'b) either = Left of 'a | Right of 'b
```

Here we have

- Type variables:    `'a`, `'b`
- Type constructor:  `either`
- Data constructors: `Left`, `Right`

Data constructors are similar to the enumerated types in other languages such as C.

The `unit` and `list` types have special syntactic sugar for their data constructors: `()` and `::` respectively.

## Records

Records allow us to give structure to data.

```ocaml
type message = { text: string; author: string; public: bool};;
let dm : message = { text = "Hello private"; author = "Joseph"; public = false };
let tweet : message = { dm with text = "Hello world!"; public = true};;
```

## Monadic/applicative types

Have a type type `'a t` where `x t` in some sense 'carries' a value or values of type `x`.

**Q1:**
So, for example, `'a list` where the list carries type `list` of type `'a`, basically parametric polymorphism?

### Map

Function

```ocaml
map : 'a t -> f:('a -> 'b) -> 'b t
```

transforms the value(s) that it carries.

Think of monads as boxes, the same as collections and lists in some ways.

For example, replace `t` with a list.

```ocaml
map : 'a list -> f:('a -> 'b) -> 'b list
```

This is a function that maps items of type `'a` to type `'b`

### % syntax

```ocaml
let%map foo = x in bar
```

translates to

```ocaml
Let_syntax.map x ~f:(fun foo -> bar)
```

**Q2:**

So if `x` was a list, then this would be the same as

```ocaml
List.map x ~f:(fun foo -> bar)
```

?

```ocaml
match%map x with
| Foo y -> bar
```

translates to

```ocaml
Let_syntax.map x ~f:(function | Foo y -> bar)
```

**Q3:** There is still a lot of the syntax that I'm ignorant of and I don't know what's pseudocode or not.
Why does the `match%map` example use capital `Foo` and the `let%map` example use lowercase?
Why does the `match%map` use `function` and the `let%map` use fun?   What is the difference?

**Q4:** >  these maps and binds are operating on the values contained by the 'deferred' async computations of type _ Async.Deferred.t

I need to read about `deferred` and async computationsin OCaml! :-)

**Q5:** Backtick

```ocaml
Deferred.repeat_until_finished () (fun () ->
  try
    ...
    let%map ... in
    `Repeat ()
  with Yojson.End_of_input -> return (`Finished ()))
```

So 'Repeat () and 'Finished () have a special backtick, like 'Assoc.
So `('Finished ()))` evaluates to the carried value?

**Q6:** There is no `~f:(...)` here

```ocaml
let%map response =
  Graphql_client.query_exn
    (Graphql_queries.Send_rosetta_transaction.make
       ~transaction:transaction_json ())
     graphql_endpoint
in
```

This seems like a slightly different form of the `let%map` syntax given earlier.

> Arg module, they have overridden the constructors for lists, so that [] and ( :: ) construct values of a different type instead.

Ouch!

### Bind

Next, consider function

```ocaml
bind : 'a t -> f:('a -> 'b t) -> 'b t
```

that uses the carried value(s) to create more carriers and combines them back together.

Think of of this bob has a box `t` of money `'a` and you `f` accept money and sell a box of books `'b t`

```ocaml
money box -> f(money -> books box) -> books box
```

Given employment contract

```ocaml
employment contract -> money
```

Function "from employment contract get BT to fix full-fiber in 3 months time"

```ocaml
money -> bt contract
```

Map way

```ocaml
employment contract -> f:(employment -> bt contract) -> bt contract contract
```

Bind way

```ocaml
employment contract -> f:(employment -> bt contract) -> bt contract
```

## Modules

Every piece of code resides in a module.  The name of the module is defined by the name of the file containg the code.

If you access the members of a module frequently you may want to make its contents accessible directly.

```ocaml
Open My_module
```

To avoid name clashes you can optionally alias your module to go by another name locally.

```ocaml
Open My_module = My
```

A Module can provide
- Functions
- Types
- Submodules

By default eveything inside a module is accessible from the outside.

To only expose what is desired you can use a module interface-- see the Interface Seciton.

## Submodules



## Interfaces

This section describes interfaces, signatures and abstract types.

**An example**

A module named `blab.ml` may have the following contents.

```ocaml
let msg = "Blah blah"

let tweet() = print_endline msg;;
```

Suppose we want to protect other modules from access `msg`.  We do this by defining an interface.

```ocaml
val msg : string

val tweet() : unit -> unit
(** Display a message **)
```

The interface is saved as `blab.mli`.

The .mli files are a natural place for documentation.  Starting comments with a
double asterisk to cause them to be picked up by the `ocamldoc` tool when generating API documentation.

Interface files are compiled before the module file using `ocamlc`, even if the module is being compiled with `ocamlopt` to machine native code.

### Signatures

Interfaces are described with signatures.

**Details**

To specify the values in an interface we used `val` declarations. The syntax of a val declaration is as follows:

```val <identifier> : <type>```


**Another example**

For example, module `Foo`

```ocaml
open Core

let bar l a =
  List.map l ~f:(x -> x + a)
```

could have an inteface file like this.

```ocaml
open Core

(** Bump the frequency count for the given string. *)
val bar : int list -> int -> int list
```

### Abstract types

Abstract types are used to provide encapsulation and prevent invariant violations.

#### Encapsulation

In the above section on interfaces we saw how functions can be exported through an interface, but what if we wanted to interface to expose an abstract type so that we are not stuck with a specific one?

When specifying a signature in an interface the type may be either
- Explicit (copied in to the interface)
- Abstract (only it's name): `type tweet`
- Omitted (completely omitted from the signature)
- Record fields made read-only: `type tweet = private {...}`

The focus of this section is on abstract types.  In this case, users of the module can manipulate objects of the type using the functions provided by the interface, but cannot access the record's fields directly.

**Example**

If we wanted to abstract away the representation of the int list we could use

```ocaml
open Core

(** A collection of ints *)
type t

(** The empty set of ints  *)
val empty : t

(** Adjust the ints with a given value *)
val bar : t -> int -> t

(** Converts the set of ints to a list of int *)
val to_list : t -> int list
```

Next we must re-write `Foo.ml` to match the interface

```ocaml
open Core

type t = int list

let empty = []

let to_list x = x

let bar l a =
  List.map l ~f:(x -> x + a)
```

Another implementation of `Foo.ml` could be

```ocaml
open Core

type t = (int,int,int.comparator_witness) Map.t

let empty = Map.empty (module Int)

let to_list t = Map.to_alist t

let bar m a =
  Map.map m ~f:(fun x -> x + a)
```

#### Preventing invariant violations



## utop

The utop command is a universal interactive toplevel for OCaml.
You can use it to interatively experiment with code, load libraries and debug modules.

Here are some useful tricks.

### printing

```ocaml
$ dune utop --profile=mainnet src/lib/signature_lib
utop # let pp fmt x =
  Format.pp_print_string fmt (Snark_params.Tick.Field.to_string x);;
val pp : Format.formatter -> Marlin_plonk_bindings_pasta_fp.t -> unit = <fun>
utop # #install_printer pp;;
utop # Hash_prefix_states.signature;;

utop # let pp fmt x =
  Format.pp_print_string fmt (Mina_numbers.Length.to_string x);;
utop # #install_printer pp;;
```

### UInt32

```ocaml
#require "integers";;
open Unsigned;;
```

## Useful modules

```ocaml
open Core;;
open Mina_numbers;;
```

## Libraries

Here are some useful libraries.

### Show

```ocaml
#require "ppx_deriving.show";;
open Core

type person = { name: string ; age: int ; human : bool }
[@@deriving show]

let () =
  let human_one = { name = "david" ; age = 32 ; human = false } in
  printf "%s\n" (show_person human_one)
```
