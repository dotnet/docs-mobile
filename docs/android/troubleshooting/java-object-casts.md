---
title: "Troubleshoot InvalidCastException when casting Java objects"
description: "This article explains why casting a Java object to another bound Java type can throw InvalidCastException in .NET for Android, often only in Release builds, and how to fix it with JavaCast<T>()."
ms.date: 11/13/2026
---

# Troubleshoot InvalidCastException when casting Java objects

A C# cast between bound Java types can fail with
`System.InvalidCastException` even though the underlying Java object
implements the requested type. The cast often works in a `Debug` build
and fails only in a `Release` build. The same root cause makes the `is`
and `as` operators return `false` and `null`, which can silently take
the wrong code path instead of throwing.

To convert between Java types, use the `JavaCast<T>()` extension method
instead of a C# cast.

## Symptoms

Consider code that unwraps the adapter of a `ListView` that has header
or footer views. In that case Android wraps the adapter in an
`android.widget.HeaderViewListAdapter`, which implements
`android.widget.WrapperListAdapter`:

```csharp
using Android.Widget;

IListAdapter? adapter = listView.Adapter;
IListAdapter? wrapped = ((IWrapperListAdapter) adapter!).WrappedAdapter;
```

This code can run correctly in a `Debug` build and then throw an
exception similar to the following in a `Release` build. The exact message
depends on the runtime that the application uses:

```console
System.InvalidCastException: Specified cast is not valid.
```

## Cause

Every Java object that crosses into managed code is represented by a
managed *peer* object. The peer's .NET type is chosen by the runtime
when the Java object is first surfaced to managed code, and it isn't
necessarily a binding for the object's actual Java class:

- The runtime uses the most derived managed binding that it can find for
  the object's Java class or one of its Java base classes, as long as
  that binding is compatible with the declared type of the API that
  returned the object.
- If no compatible binding is available, the runtime creates a peer of
  the declared type. When the declared type is an interface, such as
  `IListAdapter`, the peer is an internal *invoker* type that implements
  only that interface and the interfaces it inherits.

A C# cast, `is`, or `as` only checks the managed type of the peer. It
doesn't ask Java whether the underlying object is an instance of the
requested type. In the previous example, if the runtime created a peer
that only implements `IListAdapter`, the cast to `IWrapperListAdapter`
fails even though the Java object implements
`android.widget.WrapperListAdapter`.

Whether a more specific binding is available can depend on the build
configuration. By default, `Release` builds of Android applications
enable trimming, while `Debug` builds don't. Trimming removes bindings,
such as `Android.Widget.HeaderViewListAdapter`, that your code doesn't
reference. The same cast can therefore succeed in `Debug` because the
peer is created as `HeaderViewListAdapter`, and fail in `Release`
because that binding was removed and a less specific peer was created
instead.

The same failure can occur in any build configuration when the Java
class has no managed binding at all, for example when an API returns an
instance of a non-public Java implementation class.

## Solution

Use the `JavaCast<T>()` extension method from the `Android.Runtime`
namespace to convert between Java types:

```csharp
using Android.Runtime;
using Android.Widget;

IListAdapter? adapter = listView.Adapter;
IListAdapter? wrapped = adapter.JavaCast<IWrapperListAdapter>()?.WrappedAdapter;
```

`JavaCast<T>()` behaves as follows:

- It returns `null` when the instance is `null`.
- It returns the existing peer when the peer's managed type already
  implements or derives from `T`.
- Otherwise, it checks the type of the underlying *Java* object and, if
  that object is an instance of the Java type bound by `T`, returns a
  peer of type `T` that refers to the same Java object.
- It throws `InvalidCastException` when the underlying Java object
  isn't an instance of the Java type bound by `T`.

If the Java object might not implement the requested type, catch
`InvalidCastException` and handle that case explicitly.

`T` must be a type that binds a Java type, such as a `Java.Lang.Object`
subclass or a bound Java interface. Use C# casts and conversions for
types that don't represent Java types.

> [!TIP]
> When code needs to cast an object returned by a Java API to a different
> Java type, prefer `JavaCast<T>()` over a C# cast even if the C# cast
> currently works. Changes to trimming settings, to the bindings in use,
> or to the Android version that returns the object can change the type
> of the managed peer that the runtime creates.

## See also

- [Trimming options](/dotnet/core/deploying/trimming/trimming-options)
- [Build properties](../building-apps/build-properties.md)
