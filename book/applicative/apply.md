## Apply

We can define such a function in terms of `Maybe` directly:

```haskell
apply :: Maybe (a -> b) -> Maybe a -> Maybe b
apply (Just f) (Just x) = Just (f x)
apply _ _ = Nothing
```

If both the function and the value are present, pull them out, apply the
function, and wrap the result in `Just`. If either are missing, return `Nothing`
directly.

Here's how we can use it:

```haskell
userFromParams :: Params -> Maybe User
userFromParams params = apply
    (fmap User (getParam "name" params))
    (getParam "email" params)
```

We can make this read a bit better by taking advantage of a special syntax that
Haskell allows: if you surround a function with backticks, you can place it
*between* it's arguments rather than in front of them. This is known as *infix*
(vs *prefix* or *postfix*) notation:

```haskell
userFromParams :: Params -> Maybe User
userFromParams params =
    fmap User (getParams "name" params) `apply` getParam "email" params
```

With a bit more effort, we could end up with the following:

```haskell
userFromParams :: Params -> Maybe User
userFromParams params =
    User `fmap` getParams "name" params `apply` getParam "email" params
```

Making this form compile requires assigning a certain *fixity* to `fmap` and
`apply` so that Haskell knows which values are to be taken as arguments to which
functions. We won't go any further down that route, as it'd be wasted effort.
Instead, we'll now shift from inventing things ourselves to looking at what's
already been invented -- you'll see it ends up even better than the above.