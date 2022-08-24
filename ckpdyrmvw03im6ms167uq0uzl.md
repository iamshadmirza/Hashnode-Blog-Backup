## TIL: Difference between Omit and Exclude in TypeScript

I was trying to use `Omit` utility type to create a mapped type and it was giving me an error. Please consider the example below.

@iamshadmirza 

<details>
<summary>I am testing something on prod</summary>
## Please ignore this, thanks
Something `code` block
</details>

## The Problem

I have a type with union literals like this

```ts
type something = 'abc' | 'bcd' | 'cde' | 'def';
```

I wanted to create a Mapped type like this

```ts
type mappedType = {
    [k in something]: string;
}
```

This is equivalent to 

```ts
type mappedType = {
  abc: string;
  bcd: string;
  cde: string;
  def: string;
}
```

This works fine but let's consider if I want to omit one property.

```ts
type mappedTypeWithOmit = {
    [k in Omit<something, "def">]: string;
}
```

The above code doesn't work. Whereas if I use `Exclude` instead of `Omit`, I am able to get the desired result:

```ts
type mappedTypeWithExclude = {
    [k in Exclude<something, "def">]: string;
}
```

This is strange. I went ahead to check Omit declaration in the typescript repo and saw that it uses Exclude under the hood. 

```ts
/**
 * Construct a type with the properties of T except for those in type K.
 */
type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
```

[Click to see code in typescript repo](https://github.com/microsoft/TypeScript/blob/663b19fe4a7c4d4ddaa61aedadd28da06acd27b6/tests/baselines/reference/mappedTypeUnionConstraintInferences.js#L2)

The error I was getting was not helpful either.

```
Type 'Omit<something, "def">' is not assignable to type 'string | number | symbol'.
  Type 'Omit<something, "def">' is not assignable to type 'string'.
```

You can also check this in [TypeScript Playground](https://www.typescriptlang.org/play?#code/C4TwDgpgBAzg9gWwsAFgSwHYHMoF4oDkAhgEYDGBUAPoeQCaU0Fl0SOGsBmBA3AFB8A9IKio0MKAHc4AJwDWMPqEhQiMmURABhFBDJy8UAN58oZqAG0DmWImTpsAXQBcsYDMxZ+AXyEiAklJEGMCicFCIaKFwGNBgMnCQMqBSuhhQZDIQRMCeESQAVnqhyhACwqLoEnRwEDAYBMBK4NAIRGCQdAAqLQDqUSgA8ghRhibmltbpw1EAPPBIYtgANFAARFxrAHwubh7YPuUiYhLAcOHS8hLBdFBYaABudbBESFAkEChED2hwAK4yVQSGZNUpQNodCDdPoDACiAA8yAAbP6sMamcxWKA2BHI1EQeZ2JZYVYbCCcba7GDuTyHMxAA)

## Let's look at what I was doing wrong.

Omit is used on interface or object type and we are trying to use it on union string literal. That's why the error.

Usage:

```ts
type mappedType = {
  abc: string;
  bcd: string;
  cde: string;
  def: string;
}

type mappedTypeWithOmit = Omit<mappedType,'def'>

```
This will be equivalent to

```ts
type mappedTypeWithOmit = {
  abc: string;
  bcd: string;
  cde: string;
}
```

Exclude is different, it is used to exclude a union type.

Usage:

```ts
type something = 'abc' | 'bcd' | 'cde' | 'def';
type somethingWithExclude = Exlude<something, "def">

```
This will be equivalent to

```ts
type somethingWithExclude = 'abc' | 'bcd' | 'cde';
```

Now we can go ahead and use this to create a mapped type.

```ts
type mappedTypeWithExclude ={
  [k in somethingWithExclude]: string;
}
```

And it will be equivalent to

```ts
type mappedTypeWithExclude = {
  abc: string;
  bcd: string;
  cde: string;
}
```
Tada 🎉

## Takeaway

- Omit utility type works on object type or interfaces to omit one of the key-value pair.
- Exclude only works on union literal to exclude one of the property.
- Omit uses Pick and Exclude under the hook.

## Further Reading
- Utility Types in TypeScript: https://www.typescriptlang.org/docs/handbook/utility-types.html

I hope this helps the next time you're using Omit or Exclude.