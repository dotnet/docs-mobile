---
title: "Generic subclasses of NSObject"
description: "Learn about the limitations of generic NSObject subclasses and exported members in .NET for iOS, tvOS, Mac Catalyst and macOS."
ms.date: 10/19/2026
---

# Generic subclasses of NSObject

In .NET for iOS, tvOS, Mac Catalyst and macOS, you can use generics in subclasses of `Foundation.NSObject`. For example, on platforms that support UIKit, you can create a generic subclass of `UIView`:

```csharp
using System;
using CoreGraphics;
using UIKit;

class Foo<T> : UIView
{
    public Foo (CGRect frame) : base (frame)
    {
    }

    public override void Draw (CGRect rect)
    {
        Console.WriteLine ("T: {0}. Type: {1}", typeof (T), GetType ().Name);
    }
}
```

Because `NSObject` subclasses are registered with the Objective-C runtime, there are limitations on how generic subclasses can interact with Objective-C. These limitations apply to members exposed to Objective-C, not to ordinary managed use of generics.

## Generic type arguments in member signatures

A generic type parameter used directly as a parameter or return type in a member exposed to Objective-C must have an `NSObject` constraint.

For example, the following declaration is supported:

```csharp
using Foundation;

class Generic<T> : NSObject where T : NSObject
{
    [Export ("myMethod:")]
    public void MyMethod (T value)
    {
    }
}
```

The registrar can expose the selector `myMethod:` to Objective-C because `T` will always be `NSObject` or a subclass of it.

The following declaration is **not supported**:

```csharp
using Foundation;

class Generic<T> : NSObject
{
    [Export ("myMethod:")]
    public void MyMethod (T value)
    {
    }
}
```

Without a constraint, the registrar can't create an Objective-C signature for the exported member: its representation could differ depending on `T`.

An unconstrained type parameter is allowed when it doesn't participate in the exported signature. For example:

```csharp
using Foundation;

class Generic<T> : NSObject
{
    public T Storage { get; }

    public Generic (T value)
    {
        Storage = value;
    }

    [Export ("myMethod:")]
    public void MyMethod (NSObject value)
    {
    }
}
```

You can also mix constrained and unconstrained type parameters, as long as the parameters used in the exported signature have the required constraint:

```csharp
using System;
using Foundation;

class Generic<T, U> : NSObject where T : NSObject
{
    [Export ("myMethod:")]
    public void MyMethod (T value)
    {
        Console.WriteLine (typeof (U));
    }
}
```

Here, `T` is constrained to `NSObject`, and `U` isn't part of the exported signature.

## Closed generic wrapper types in exported signatures

Deriving a generic wrapper from `NSObject` doesn't make every constructed version of that wrapper valid in an exported signature. In particular, a wrapper whose generic argument is an enum, another value type, or a managed type that doesn't implement `ObjCRuntime.INativeObject` isn't supported in that signature, even though the wrapper itself is an `NSObject`-derived reference type.

For example, the following property is **not supported**:

```csharp
using System;
using Foundation;

public enum MyEnum
{
    A
}

public class EnumWrapper : NSObject
{
    public static EnumWrapper<T> Create<T> (T value) where T : Enum
    {
        return new EnumWrapper<T> (value);
    }
}

public class EnumWrapper<T> : EnumWrapper where T : Enum
{
    public T Value { get; }

    public EnumWrapper (T value)
    {
        Value = value;
    }
}

public class Example : NSObject
{
    [Export ("Test")]
    public EnumWrapper<MyEnum> Test { get; } = EnumWrapper.Create (MyEnum.A);
}
```

The `where T : Enum` constraint doesn't satisfy the native-object requirement. The same limitation affects wrappers such as `Generic<int>` and `Generic<object>` in exported signatures. Closed generic types with supported native-object type arguments aren't subject to this limitation.

> [!IMPORTANT]
> Older static registrars rejected the enum-wrapper example with MT4166 or MM4166, as reported in [dotnet/macios#15709](https://github.com/dotnet/macios/issues/15709). Newer static registrars and the dynamic registrar can accept it without a diagnostic, but the signature remains unsupported. The planned .NET 12 fix adds registrar checks to reject this signature; it doesn't add support for it. Don't rely on a particular registrar or build configuration accepting this code. Use a non-generic derived wrapper instead.

### Export a non-generic derived wrapper

Keep the generic wrapper for managed code, but expose a concrete, non-generic subclass in the exported signature instead. The following is a complete alternative to the preceding example:

```csharp
using System;
using Foundation;

public enum MyEnum
{
    A
}

public class EnumWrapper : NSObject
{
    public static MyEnumWrapper Create (MyEnum value)
    {
        return new MyEnumWrapper (value);
    }
}

public class EnumWrapper<T> : EnumWrapper where T : Enum
{
    public T Value { get; }

    public EnumWrapper (T value)
    {
        Value = value;
    }
}

public class MyEnumWrapper : EnumWrapper<MyEnum>
{
    public MyEnumWrapper (MyEnum value) : base (value)
    {
    }
}

public class Example : NSObject
{
    [Export ("Test")]
    public MyEnumWrapper Test { get; } = EnumWrapper.Create (MyEnum.A);
}
```

Both the property type and the factory return type are `MyEnumWrapper`, not `EnumWrapper<MyEnum>`. The derived wrapper's constructor forwards the enum value to the generic base constructor. In this example, wrappers are created in managed code; creating managed wrappers for native instances is a separate concern, discussed next.

## Instantiating generic types from Objective-C

Objective-C can't instantiate a generic managed type directly. This commonly arises when a managed class is used in a xib or storyboard.

Consider the following declaration. In .NET, the `NativeHandle` constructor is used to construct a managed wrapper for a native Objective-C instance:

```csharp
using Foundation;
using ObjCRuntime;

class Generic<T> : NSObject where T : NSObject
{
    public Generic ()
    {
    }

    protected Generic (NativeHandle handle) : base (handle)
    {
    }
}
```

The declaration is valid for managed use. However, if Objective-C tries to create an instance of the generic type, the runtime throws an exception: Objective-C has no way to specify the type argument.

Use a non-generic subclass with a fixed type argument instead. For example, on platforms that support UIKit:

```csharp
using Foundation;
using ObjCRuntime;
using UIKit;

class Generic<T> : NSObject where T : NSObject
{
    public Generic ()
    {
    }

    protected Generic (NativeHandle handle) : base (handle)
    {
    }
}

[Register ("GenericUIView")]
class GenericUIView : Generic<UIView>
{
    public GenericUIView ()
    {
    }

    protected GenericUIView (NativeHandle handle) : base (handle)
    {
    }
}
```

The class `GenericUIView` can be used in a xib or storyboard because its type argument is fixed.

## Exported generic methods aren't supported

You can't export a generic method to Objective-C. The following declaration is **not supported** and is rejected by the registrar:

```csharp
using Foundation;

class MyClass : NSObject
{
    [Export ("myMethod:")]
    public void MyMethod<T> (T argument)
    {
    }
}
```

The runtime doesn't know which type to use for `T` when Objective-C invokes the method.

Instead, export a non-generic method that calls the generic method with a specific type argument:

```csharp
using Foundation;
using UIKit;

class MyClass : NSObject
{
    [Export ("myMethod:")]
    public void MyUIViewMethod (UIView argument)
    {
        MyMethod<UIView> (argument);
    }

    public void MyMethod<T> (T argument)
    {
    }
}
```

## Exported static members in generic types aren't supported

You can't expose a static member to Objective-C if it's declared in a generic subclass of `NSObject`, even when the member doesn't use the type parameter. For example, the following declarations are **not supported**:

```csharp
using Foundation;

class Generic<T> : NSObject where T : NSObject
{
    [Export ("myMethod")]
    public static void MyMethod ()
    {
    }

    [Export ("myProperty")]
    public static T MyProperty { get; set; }
}
```

For an instance member, the runtime can determine the type argument from the instance: it will always be something like `Generic<UIView>`, never the open type `Generic<T>`. There is no instance to provide that information for a static member.

Instead, export static members from a non-generic subclass:

```csharp
using Foundation;
using UIKit;

class GenericUIView : Generic<UIView>
{
    [Export ("myUIViewMethod")]
    public static void MyUIViewMethod ()
    {
        MyMethod ();
    }

    [Export ("myProperty")]
    public static UIView MyUIViewProperty
    {
        get { return MyProperty; }
        set { MyProperty = value; }
    }
}

class Generic<T> : NSObject where T : NSObject
{
    public static void MyMethod ()
    {
    }

    public static T MyProperty { get; set; }
}
```

## Performance

The static registrar can't resolve an exported member declared in a generic type at build time in the same way as a member in a non-generic type. It must look up the member at runtime, so invoking such a member from Objective-C has additional lookup overhead.
