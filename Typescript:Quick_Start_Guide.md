# TypeScript: Quick Start Guide

## Basics

```typescript
let isDone: boolean = false;
let lines: number = 42;
let name: string = "Anders";
let notSure: any = 4;

let list: number[] = [1, 2, 3];
let list: Array<number> = [1, 2, 3];

enum Color { Red, Green, Blue };
let c: Color = Color.Green;

// Tagged Union Types
type State =
  | { type: "loading" }
  | { type: "success", value: number }
  | { type: "error", message: string };

declare const state: State;
if (state.type === "success") {
  console.log(state.value);
} else if (state.type === "error") {
  console.error(state.message);
}

// Type Assertion
let foo = {}
foo.bar = 123 // Error
foo.baz = 'hello world' // Error

interface Foo {
  bar: number;
  baz: string;
}

let foo = {} as Foo; // Type assertion here
foo.bar = 123;
foo.baz = 'hello world'

// "void" is used in the special case of a function returning nothing
function bigHorribleAlert(): void {
  alert("I'm a little annoying box!");
}

// The following are equivalent, the same signature will be inferred by the compiler, and same JavaScript will be emitted
let f1 = function (i: number): number { return i * i; }

// Return type inferred
let f2 = function (i: number) { return i * i; }

// "Fat arrow" syntax
let f3 = (i: number): number => { return i * i; }

// "Fat arrow" syntax with return type inferred
let f5 = (i: number) => { return i * i; }

// "Fat arrow" syntax with return type inferred, braceless means no return keyword needed
let f5 = (i: number) => i * i;
```
## Interfaces

```typescript
// Interfaces are structural, anything that has the properties is compliant with the interface
interface Person {
  name: string;
  age?: number; // Optional properties, marked with a "?"
  move(): void; // And of course functions
}

let p: Person = { name: "Bobby", move: () => { } };
let validPerson: Person = { name: "Bobby", age: 42, move: () => { } };
let invalidPerson: Person = { name: "Bobby", age: true }; // Is not a person because age is not a number

// Interfaces can also describe a function type
interface SearchFunc {
  (source: string, subString: string): boolean;
}
// Only the parameters' types are important, names are not important.
let mySearch: SearchFunc;
mySearch = function (src: string, sub: string) {
  return src.search(sub) != -1;
}
```
## Classes

```typescript

class Point {
  x: number; // members are public by default

  constructor(x: number, public y: number = 0) {
    this.x = x;
  }

  dist() { return Math.sqrt(this.x * this.x + this.y * this.y); }

  static origin = new Point(0, 0);
}

class PointPerson implements Person {
    name: string
    move() {}
}

let p1 = new Point(10, 20);
let p2 = new Point(25); //y will be 0

// Inheritance
class Point3D extends Point {
  constructor(x: number, y: number, public z: number = 0) {
    super(x, y); // mandatory
  }
  // Overwrite
  dist() { }
}
```

## Generic

```typescript
class Tuple<T1, T2> {
  constructor(public item1: T1, public item2: T2) { }
}

interface Pair<T> {
  item1: T;
  item2: T;
}

let pairToTuple = function <T>(p: Pair<T>) {
  return new Tuple(p.item1, p.item2);
};

let tuple = pairToTuple({ item1: "hello", item2: "world" });
```

## Readonly: TypeScript 3.1
```typescript
let moreNumbers: ReadonlyArray<number> = numbers;

interface Person {
  readonly name: string;
  readonly age: number;
}

class Car {
  readonly make: string;
  readonly model: string;
  readonly year = 2018;

  constructor() {
    this.make = "Unknown Make"; // Assignment permitted in constructor
    this.model = "Unknown Model"; // Assignment permitted in constructor
  }
}
```

