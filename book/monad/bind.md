## Bind

If you haven't guessed it, Haskell does have exactly this function. Its name is
*bind* and it's defined as part of the `Monad` type class. Here is its type
signature:

```haskell
(>>=) :: m a -> (a -> m b) -> m b
-- 
-- where m is the type you're saying is a Monad (e.g. Maybe)
-- 
```

Again, you can see that `andThen` has the correct signature:

```haskell
--         m     a    (a -> m     b) -> m     b
andThen :: Maybe a -> (a -> Maybe b) -> Maybe b
```

## Chaining

`(>>=)` is defined as an operator because it's meant to be used infix. It's also
given an appropriate fixity so it can be chained together intuitively. This is
why I chose the name `andThen` for my fictitious version: it can sometimes help
to read `x >>= y >>= z` as *x and-then y and-then z*. To see this in action,
let's walk through another example.

Suppose we are working on a system with the following functions for dealing with
users' addresses and their zip codes:

```haskell
-- Returns Maybe because the user may not exist
findUser :: UserId -> Maybe User
findUser = undefined

-- Returns Maybe because Users aren't required to have an address on file
userZip :: User -> Maybe ZipCode
userZip = undefined
```

Let's also say we have a function to calculate shipping costs by zip code. It
employs `Maybe` to handle invalid zip codes:

```haskell
shippingCost :: ZipCode -> Maybe Cost
shippingCost = undefined
```

We could naively calculate the shipping cost for some user given their Id:

```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid =
    case findUser uid of
        Just u -> case userZip u of
            Just z -> case shippingCost z of
                Just c  -> Just c

                -- User has an invalid zip code
                Nothing -> Nothing

            -- Use has no address
            Nothing -> Nothing

        -- User not found
        Nothing -> Nothing
```

This code is offensively ugly, but it's the sort of code I write every day in
Ruby. We might hide it behind three-line methods each holding one level of
conditional, but it's there.

How does this code look with `(>>=)`?

```haskell
findUserShippingCost :: UserId -> Maybe Cost
findUserShippingCost uid = findUser uid >>= userZip >>= shippingCost
```

You have to admit, that's quite nice. Hopefully even more so when you look back
at the definition for `andThen` to see that that's all it took to clean up this
boilerplate.
