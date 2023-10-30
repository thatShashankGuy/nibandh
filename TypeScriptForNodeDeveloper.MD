# TypeScript Guide for Node Developers

## Introduction

By now, TypeScript has emerged as a fully-fledged de facto standard for writing secure, enterprise-grade Node apps. This guide draws from both my experience and the TypeScript handbook to offer a brief introduction for those transitioning from JavaScript. Readers should have some familiarity with JavaScript and Object-Oriented Programming (OOP). While I've strived for conciseness, I've also attempted to delve as deep as necessary, offering numerous code examples. For a deeper dive into any topic, resources are provided at the end.

## TypeScript as a Superset of JavaScript

All JavaScript code is valid TypeScript. This means you can create a TypeScript file using pure JavaScript and then compile that script using the TypeScript compiler (`tsc`). 

For our exercises, we'll utilize the [TypeScript playground](https://www.typescriptlang.org/play).

Consider this JavaScript code:

```javascript
function func1(a) {
	let b = 7;
	return b + a;
} 
let c = func1(7);
console.log(c); // 14
c = func1('s');
console.log(c); // 7s
```

When you compile this JavaScript code in the playground, it should compile without issues.

## Writing Your First TypeScript Code

Let's transition the JavaScript example above to TypeScript by introducing types:

```typescript
function func1(a: number): number {
	let b = 7;
	return b + a;
} 
let c = func1(7);
console.log(c); // 14
// c = func1('s');  This would raise an error
```

In TypeScript, we use the `:` notation to denote types. Here are some foundational TypeScript types:

```typescript
let a: number;
let b: string;
let c: boolean;
let d: [];  // array
let e: {};  // object
let f: unknown;
let g: never;
let h: any;
let i: null;
let j: undefined;
let k: void;
```

We'll further explore `any`, `unknown`, and `never` later in this guide.

## Type Safety and Type Inference

Here's the syntax for explicitly defining types:

```typescript
let a: number;
```

Once a type is declared, TypeScript enforces constraints related to data type assignments:

```typescript
a = 6;       // Valid
// a = 'six';  This would be flagged
```

If types aren't explicitly declared, TypeScript uses type inference to identify and then enforce the type:

```typescript
let a = 6;           // TypeScript infers that a is of type number
// a = 'six';       This would result in an error
a = 78;              // Valid
```

## Return Types

Functions in TypeScript can also be associated with return types, which help in indicating the kind of data a function returns. Given that functions are first-class citizens in JavaScript, associating them with a return type is essential. 

We can also infer return types, which means that if the function's return type is obvious from the code, TypeScript will automatically determine it. However, explicitly defining return types can enhance readability and reduce potential errors.

Here's an example:

```typescript
function func1(a: number): number {
	let b = 7;
	return b + a;
}
console.log(typeof func1(7)); // Outputs: number
```

## Advanced Types: `any`, `unknown`, and `never`

### The `any` Type

The `any` type in TypeScript denotes that a variable could essentially be of any data type. It's like writing plain JavaScript without the benefits of TypeScript's type-checking:

```typescript
let variable: any = 'string';
variable = 7;
variable = true;
```

While it offers flexibility, overuse of `any` can defeat the purpose of using TypeScript. It's best to use it judiciously.

### The `unknown` Type

The `unknown` type is a type-safe counterpart to `any`. It allows anything to be assigned to it, but to use it further, you'll need to provide type checks or type assertions.

Here's a simple distinction between `any` and `unknown`:

```typescript
let vAny: any = 10;
let vUnknown: unknown = 10;

let s1: string = vAny;     
// let s2: string = vUnknown;  This would be flagged
```

### The `never` Type

The `never` type indicates values that will never occur. For instance:

```typescript
function throwError(errorMsg: string): never {
	throw new Error(errorMsg);
}
```

While both `void` and `never` indicate the absence of a value, they're used differently. With `void`, a function doesn't return a value. With `never`, a function will not complete normally (e.g., it will throw an error).


