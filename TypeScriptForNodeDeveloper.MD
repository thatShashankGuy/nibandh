# TypeScript Guide for Node Developers

## Introduction

By now, TypeScript has emerged as a fully-fledged de facto standard for writing secure, enterprise-grade Node apps. I am building this guid from  my experience and the TypeScript handbook to offer a brief introduction for those transitioning from JavaScript but please beware I am no SME on typescript. Readers should have some familiarity with JavaScript and Object-Oriented Programming (OOP). As we all are busy people I have kept the guide as concise as possible while being as clear as  I can. For a deeper dive into any topic, resources are provided at the end.

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

While it offers flexibility, overuse of `any`  defeats the purpose of using TypeScript. It's best to use it sparsely.

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

## Unions 

Unions are way to combine multiple primitive type to form  a custom type .

Consider a system  which emits a signal in either true and false  or 0 and 1. If you want to store the data in a variable but you are not sure what will it emit next so you handle you function.

```typescript
let a = emit() // true 
a = emit() // this time it emits 1 and your typescript will throw erorr  because of the inference typescript have assinged a the type of boolean 
```
In this scenario you can create a `union` of boolean and Number type so that variable can store either  number or boolean 

```typescript
let a  : boolean | number
a = true // valid 
a = 1 // valid 
```

### Literal Types 

But we can optimize this as well as we know that system will only emit 1s and 0s so no need to create an union with Number   you can create union with `Literal Types`. Literal Types as name suggest enforce storing only a particular value or a handful of values in case of union . Lets update our function 

```typescript
let a : boolean | 1 | 0
a = 1 //valid 
a = true // valid 
a = false //valid 
a = 0 // valid 
a = 2 // invalid - typescript will throw errror 
```
you can use Literal types and Unions for custom checkers 

```typescript
let checker : undefined | null | "" | 0 | false
```

### Narrowing 
Consider a function called  handleSyscall which takes an argument x  provided at runtime  , x can either have data of type string or it could be undefined . so we define res with type string | undefined . We want to return syscall response with message all lowercase. Lets write code forthat  

```typescript 
function handleSysCall(x : string | undefined){
    return  x.toLowerCase() // Typescript throws a error - > 'x' is possibly 'undefined'.
}

let x = await syscall()
handleSysCall(x)
```
Notice that we dont know the type of x at the run time we cannot run string operation as there is a good chance x will be undefined 
To handle this in  typescript  we perform  `Narrowing`

In simple terms, dynamic checks and predicates provide the information about values at runtime, So, Type narrowing is the process of providing this information during the compile time.

TypeScript follows some possible paths of execution that the programs can take to analyze the most specific possible type of a value at a given position. It searches for some special checks known as Type Guards and assignments. The process of refining types to some more specific types is known as Narrowing.

lets perform Narrowing by checking type of x at runtime to make sure that we do not encounter an error due to type mismatch


```typescript 
function handleSysCall(x : string | undefined){
    if(x !== undefined) return  x.toLowerCase() //Valid  -> Type script checks type and value at runtime to return what is valid for both types . 
    return 'syscall did not return value' 
}

let x = await syscall()
handleSysCall(x)
```


