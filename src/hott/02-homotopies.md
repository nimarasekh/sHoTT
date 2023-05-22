# 2. Homotopies

This is a literate `rzk` file:

```rzk
#lang rzk-1
```
## Homotopies and their algebra

```rzk
#section homotopies

#variables A B : U

-- The type of homotopies between parallel functions.
#def homotopy 
    (f g : A -> B)          -- Two parallel functions.
    : U
    := (a : A) -> (f a = g a)

-- The reversal of a homotopy    
#def homotopy-rev 
    (f g : A -> B)          -- Two parallel functions.
    (H : homotopy f g)  -- A homotopy from f to g.
    : homotopy g f
    := \a -> rev B (f a) (g a) (H a)

-- Homotopy composition is defined in diagrammatic order like concat but unlike composition.
#def homotopy-composition 
    (f g h : A -> B)          -- Three parallel functions.
    (H : homotopy f g)
    (K : homotopy g h)
    : homotopy f h
    := \a -> concat B (f a) (g a) (h a) (H a) (K a)

#end homotopies
```

## Whiskering homotopies

```rzk
#section homotopy-whiskering

#variables A B C : U

#def homotopy-postwhisker 
    (f g : A -> B)              -- Two parallel functions.
    (H : homotopy A B f g)      -- A homotopy from f to g.
    (h : B -> C)
    : homotopy A C (composition A B C h f) (composition A B C h g)
    := \a -> ap B C (f a) (g a) h (H a)

#def homotopy-prewhisker 
    (f g : B -> C)              -- Two parallel functions
    (H : homotopy B C f g)
    (h : A -> B)
    : homotopy A C (composition A B C f h) (composition A B C g h)
    := \a -> H (h a)

#end homotopy-whiskering
```

## Naturality

```rzk
-- The naturality square associated to a homotopy and a path.
#def nat-htpy 
    (A B : U)               -- Two types.
    (f g : A -> B)          -- Two parallel functions.
    (H : homotopy A B f g)  -- A homotopy from f to g.
    (x y : A)
    (p : x = y)
    : (concat B (f x) (f y) (g y) (ap A B x y f p) (H y)) = (concat B (f x) (g x) (g y) (H x) (ap A B x y g p))
    := idJ(A, x, 
            \y' p' 
                -> (concat B (f x) (f y') (g y') 
                        (ap A B x y' f p') (H y')) =
                   (concat B (f x) (g x) (g y') 
                        (H x) (ap A B x y' g p')), 
            refl-concat B (f x) (g x) (H x), y, p)
```

## An application

```rzk
#section cocone-naturality

#variable A : U
#variable f : A -> A
#variable H : homotopy A A f (identity A)
#variable a : A

-- In the case of a homotopy H from f to the identity the previous square applies to the path H a to produce the following naturality square.
#def cocone-naturality 
    : (concat A (f (f a)) (f a) a (ap A A (f a) a f (H a)) (H a)) =
            (concat A (f (f a)) (f a) (a) (H (f a)) (ap A A (f a) a (identity A) (H a)))
    := nat-htpy A A f (identity A) H (f a) a (H a)

-- After composing with ap-id, this naturality square transforms to the following:
#def reduced-cocone-naturality 
    : (concat A (f (f a)) (f a) a (ap A A (f a) a f (H a)) (H a)) =
            (concat A (f (f a)) (f a) (a) (H (f a)) (H a))
    := concat ((f (f a)) = a) (concat A (f (f a)) (f a) a (ap A A (f a) a f (H a)) (H a)) 
            (concat A (f (f a)) (f a) (a) (H (f a)) (ap A A (f a) a (identity A) (H a)))
            (concat A (f (f a)) (f a) (a) (H (f a)) (H a))
                (cocone-naturality)
                (concat-homotopy A (f (f a)) (f a) (H (f a)) a (ap A A (f a) a (identity A) (H a)) (H a) (ap-id A (f a) a (H a)))

-- Cancelling the path (H a) on the right and reversing yields a path we need:
#def cocone-naturality-coherence 
    : (H (f a)) =(ap A A (f a) a  f (H a))  
    := rev (f (f a) = f a) (ap A A (f a) a  f (H a)) (H (f a)) 
        (concat-right-cancel A (f (f a)) (f a) a 
            (ap A A (f a) a  f (H a)) 
            (H (f a)) 
            (H a) 
                (reduced-cocone-naturality))

#end cocone-naturality                
```