In the last chapter, we defined a type that allows any value of type `a` to
carry with it additional information about whether it's actually there or not:

```haskell
data Maybe a = Nothing | Just a

actuallyFive :: Maybe Int
actuallyFive = Just 5

notReallyFive :: Maybe Int
notReallyFive = Nothing
```

As you can see, attempting to get at the value inside is dangerous:

```haskell
getValue :: Maybe a -> a
getValue (Just x) = x
getValue Nothing = error "uh-oh"

getValue actuallyFive
-- => 5

getValue notReallyFive
-- => Runtime error!
```

At first, this seems severely limiting: how can we use something if we can't
(safely) get at the value inside?

## Choices

When confronted with some `Maybe a`, and you want to do something with an `a`,
you have three choices:

1. Use the value if you can, otherwise throw an exception
2. Use the value if you can, but still have some way of returning a valid result
   if the value's not there
3. Pass the buck and return a `Maybe` result yourself

The first option is a non-starter. As you saw, it is possible to throw runtime
exceptions in Haskell via the `error` function, but you should avoid this at all
costs. We're trying to eliminate runtime exceptions, not add them.

The second option is possible only in certain scenarios. You need to have some
way to handle an incoming `Nothing`. That may mean skipping certain aspects of
your computation or substituting another appropriate value. Usually, if you're
given a completely abstract `Maybe a`, it's not possible to determine a
substitute because you can't produce a value of type `a` out of nowhere.

Even if you did know the type (say you were given a `Maybe Int`) it would be
unfair to your callers if you defined the safe substitute yourself. In one case
`0` might be best because we're going to add something, but in another `1` would
be better because we plan to multiply. It's best to let them handle it
themselves using a utility function like `fromMaybe`:

```haskell
fromMaybe :: a -> Maybe a -> a
fromMaybe x Nothing = x
fromMaybe _ (Just x) = x

fromMaybe 10 actuallyFive
-- => 5

fromMaybe 10 notReallyFive
-- => 10
```

Option 3 is actually a variation on option 2. By making your own result a
`Maybe` you always have the ability to return `Nothing` yourself if the value isn't present. If the value *is* present, you can perform whatever computation you
need to and wrap what would be your normal result in `Just`.

The main downside is that now your callers also have to consider how to deal
with the `Maybe`. Given the same situation, they should again make the same
choice (option 3), but that only pushes the problem up to their callers--which means any
`Maybe` values tend to go *viral*.

Eventually, probably at some UI boundary, someone will need to "deal with" the
`Maybe`, either by providing a substitute or skipping some action that might
otherwise take place. This should happen only once, at that boundary. Every
function between the source and the final use should pass along the value's
potential non-presence unchanged.

Even though it's safest for every function in our system to pass along a `Maybe`
value, it would be extremely annoying to force them all to actually take and
return `Maybe` values. Each function separately checking whether it should go ahead
and perform its computations will become repetitive and tedious. Instead, we
can completely abstract this "pass along the `Maybe`" concern using higher-order
functions and something called *functors*.

## Discovering a Functor

Imagine we had a higher-order function called `whenJust`:

```haskell
whenJust :: (a -> b) -> Maybe a -> Maybe b
whenJust f (Just x) = Just (f x)
whenJust _ Nothing = Nothing
```

It takes a function from `a` to `b` and a `Maybe a`. If the value's there, it
applies the function and wraps the result in `Just`. If the value's not there,
it returns `Nothing`. Note that it constructs a new value using the `Nothing`
constructor. This is important because the value we're given is type `Maybe a`
and we must return type `Maybe b`.

This allows the internals of our system to be made of functions (e.g. the `f`
given to `whenJust`) that take and return normal, non-`Maybe` values, but still
"pass along the `Maybe`" whenever we need to take a value from some source that
may fail and manipulate that value in some way. If it's there, we go ahead and
manipulate it, but return the result as a `Maybe` as well. If it's not, we
return `Nothing` directly.

```haskell
whenJust (+5) actuallyFive
-- => Just 10

whenJust (+5) notReallyFive
-- => Nothing
```

This function exists in Haskell's Prelude[^prelude] as `fmap` in the `Functor`
type class.

[^prelude]: The module of functions available without `import`ing anything.
