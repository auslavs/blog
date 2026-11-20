---
title: "First Thoughts on C# 15 Unions"
date: 2026-11-15
tags: ['dotnet', 'csharp', 'fsharp']
---

**TL;DR:** C# 15 finally has union types. They're a welcome addition, but not quite the discriminated unions I'm used to in F#. Untagged unions offer some nice compositional advantages, but introduce surprising behaviour around null. And while C# has adopted some of the benefits of TypeScript's approach, it misses out on one I've been wanting in F# for years.

After years of working in F#, I've recently moved to a C# team. So the timing of C# finally getting union types couldn't have been much better.

Discriminated unions are one of the F# features I've come to rely on, and I was keen to see what C# had come up with. I was excited, but cautiously optimistic about whether the implementation would meet my expectations.

I watched {{< new-tab-link href="https://www.youtube.com/watch?v=7qW92AwlrjM" >}}Mads Torgersen and Dustin Campbell's presentation at Build 2026{{< /new-tab-link >}}, and initially things were looking pretty good.

Then they introduced a nullable type.

Suddenly, the switch needed another case to handle `null`. Worse, if two of the union's types were nullable, there was no way to tell which one the null belonged to.

That caught me by surprise.

## Tagged vs untagged unions

The distinction comes down to how the union identifies its cases.

F# uses **tagged unions**. Each case has a name, and that name is part of the value.

```fsharp
type Contact =
  | Email of string
  | Phone of string
```

Both cases contain a string, but `Email "123"` and `Phone "123"` are different values. The tag tells us what they represent, regardless of the underlying data.

C# 15 takes a different approach with **untagged unions**.

```csharp
public union Pet(Cat, Dog);
```

Here, the cases are identified by their types. There's no separate tag. If the value is a `Cat`, it's the `Cat` case.

This is quite convenient. You can combine existing types, including ones from libraries you don't control, without introducing wrappers or a common base class.

But it also means you can't distinguish two cases containing the same type. If you want an email address and a phone number, both represented as strings, you'll need separate wrapper types.

Essentially, the type becomes the tag.

## The null problem

This brings me back to what caught me off guard in the presentation.

Consider a union containing two nullable types:

```csharp
public union MaybePet(Cat?, Dog?);

string Describe(MaybePet pet) => pet switch
{
    Cat c => c.Name,
    Dog d => d.Name,
    null => "No pet"
};
```

In this example, omitting the `null` arm produces warning CS8655 because the compiler tracks the union’s contents as possibly null.

That's reasonable enough, but which case was it? A null `Cat` or a null `Dog`?

There's no way to know. Null has no runtime type, and the union doesn't retain a separate discriminator.

In F#, the distinction survives because the tag exists independently of the payload.

There's another little surprise. C# unions declared with `union` are structs, and `default(Pet)` has a null underlying value, even when neither case type is nullable.

I'd generally avoid nullable union cases and represent absence explicitly instead.

```csharp
public record NoPet;
public union Pet(Cat, Dog, NoPet);
```

Which is starting to look suspiciously like an F# discriminated union, just with extra steps.

## Something I've been reaching for in F#

Despite the null issue, there are good reasons for C#'s approach. Being able to compose existing types without wrapping them is genuinely useful.

But there's another kind of flexibility I've found myself wanting in F#.

Suppose I have a user profile that can be in several states. I've already validated it upstream and eliminated some of those states. Now I want to pass it downstream, without making the next function handle cases that can no longer occur.

In F#, this means defining another union and explicitly converting between them.

I was hoping C#'s untagged unions might make this easier, but unfortunately they're still distinct named types.

```csharp
public union Profile(Pending, Active, Suspended);
public union Usable(Active, Suspended);

void Process(Profile profile)
{
    if (profile is Pending) return;

    Use(profile); // Doesn't compile
}

void Use(Usable profile) { }
```

Even though we've eliminated `Pending`, the compiler still considers `profile` to be a `Profile`.

TypeScript takes this further. Its unions are structural, so a smaller union is compatible with a larger one, and the compiler can narrow the possibilities as cases are eliminated.

```typescript
type Profile = "pending" | "active" | "suspended";
type Usable = "active" | "suspended";

declare function use(profile: Usable): void;

function process(profile: Profile) {
  if (profile === "pending") return;

  use(profile); // Works!
}
```

No conversion or additional wrapper required.

This is **narrowing**, and the reverse compatibility, passing a smaller union where a larger one is expected, is **widening**.

Interestingly, OCaml demonstrates that you don't actually need untagged unions to get similar flexibility.

Its *polymorphic variants* retain explicit tags but support structural compatibility.

```ocaml
let describe = function
  | `Active -> "active"
  | `Suspended -> "suspended"
```

The compiler infers that the function accepts at most those two tags. Unlike ordinary F# union cases, the tags aren't tied to a particular declared union type.

That's a pretty appealing combination of what I like about F# and TypeScript.

F# has had a proposal for [anonymous type-tagged unions (FS-1092)](https://github.com/fsharp/fslang-design/discussions/519) for years, but it hasn't shipped.

## What I do in F# today

My workaround is to give each state its own record, then define smaller unions where needed.

```fsharp
type Pending = { Email: string }
type Active = { Email: string; Plan: string }
type Suspended = { Email: string; Reason: string }

type Profile =
  | Pending of Pending
  | Active of Active
  | Suspended of Suspended

type Usable =
  | UsableActive of Active
  | UsableSuspended of Suspended

let toUsable (profile: Profile) : Usable option =
  match profile with
  | Pending _ -> None
  | Active a -> Some (UsableActive a)
  | Suspended s -> Some (UsableSuspended s)
```

Now downstream functions accept `Usable` and don't need to deal with `Pending`.

It works, it's type-safe, and the compiler ensures I handle every possible case. It's just more ceremony than I'd like.

If only one case survives, I can skip the smaller union and pass the record directly.

## Wrapping up

I think C# 15 unions are a good addition. The ability to compose existing types is useful, and the design fits naturally into C#.

The nullable behaviour caught me by surprise, but I can understand the trade-offs.

What I find more disappointing is that C# doesn't offer the structural narrowing and widening that TypeScript provides. It's something I've wanted in F# for a while, and OCaml shows it can work with tagged unions too.

Until then, more records it is.
