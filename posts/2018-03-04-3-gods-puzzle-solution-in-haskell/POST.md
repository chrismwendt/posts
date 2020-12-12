---
title: 3 gods puzzle solution in Haskell
published: true
image: https://i.imgur.com/11DYbM0.png
---

# The Puzzle

Here's the prompt from [Wikipedia][wikipedia]:

> Three gods A, B, and C are called, in no particular order, True, False, and Random. True always speaks truly, False always speaks falsely, but whether Random speaks truly or falsely is a completely random matter. Your task is to determine the identities of A, B, and C by asking three yes-no questions; each question must be put to exactly one god. The gods understand English, but will answer all questions in their own language, in which the words for yes and no are da and ja, in some order. You do not know which word means which.

# My solution

The puzzle can be simplified by eliminating redundancies masquerading as complications:

- It doesn't matter that the gods speak a different language because you can interpret "da" as "yes" by asking the god to negate their response if "da" means "no".
- It doesn't matter that one god lies because you can ask the god to negate their response if they are False.

It's clear that you can't ask each god a question because since one responds randomly, you'll only get 2 bits of information (which is only enough to enumerate 4 outcomes, and there are 6 permutations of gods). Therefore, you have to spend a question to find a non-random god to ask the remaining questions.

```
Ask A: Is B Random?
  Yes -> Ask C: Is A Random?
    Yes -> Ask C: Is C True?
      Yes -> RFT
      No -> RTF
    No -> Ask C: Is C True?
      Yes -> FRT
      No -> TRF
  No -> Ask B: Is A Random?
    Yes -> Ask B: Is B True?
      Yes -> RTF
      No -> RFT
    No -> Ask B: Is B True?
      Yes -> FTR
      No -> TFR
```

This code performs 10 sample runs with a random permutation each time (which only gives probabilistic evidence that it's correct).

```haskell
#!/usr/bin/env stack
{-
  stack
  --resolver lts-10.1
  --install-ghc
  --package random-extras
  --package random-fu
  script
-}

{-# LANGUAGE OverloadedStrings #-}

import System.Random
import Control.Monad
import Data.Random.Extras
import Data.Random.RVar
import Data.Random.Source.DevRandom

data GodType = T | F | R deriving (Eq, Show)
type GodID = Int
type Permutation = [GodType]
data PrimQ =
  PrimQ GodID GodID GodType -- (X, Y, Z): ask god X "Is god Y of type Z?"
  deriving (Eq, Show)
data Eng = Yes | No deriving (Eq, Show)
data Urk = Da | Ja deriving (Eq, Show)
data LangMap = DaMeansYes | DaMeansNo deriving (Eq, Show)
type Question = ([GodType], GodType, LangMap) -> Eng
data Strategy =
  SDone Permutation
  | SQuestion
    GodID -- who to ask
    Question
    Strategy -- when the repsonse is "da"
    Strategy -- when the repsonse is "ja"

boolToEng :: Bool -> Eng
boolToEng True = Yes
boolToEng False = No

solution :: Strategy
solution =
  let
    q about theType (types, ty, langMap) =
      boolToEng
      $ (case ty of
        T -> id
        F -> not
        R -> id)
      $ (if langMap == DaMeansYes then id else not)
      $ (types !! about) == theType
  in
    SQuestion 0 (q 1 R)
      (SQuestion 2 (q 0 R)
        (SQuestion 2 (q 2 T)
          (SDone [R, F, T])
          (SDone [R, T, F]))
        (SQuestion 2 (q 2 T)
          (SDone [F, R, T])
          (SDone [T, R, F])))
      (SQuestion 1 (q 0 R)
        (SQuestion 1 (q 1 T)
          (SDone [R, T, F])
          (SDone [R, F, T]))
        (SQuestion 1 (q 1 T)
          (SDone [F, T, R])
          (SDone [T, F, R])))

sampleRun :: Strategy -> IO ()
sampleRun s = do
  perm <- runRVar (shuffle [T, F, R]) DevRandom
  langMap <- fmap ([DaMeansYes, DaMeansNo] !!) $ randomRIO (0, 1)

  let
    flipEng Yes = No
    flipEng No = Yes

    engToUrk DaMeansYes Yes = Da
    engToUrk DaMeansYes No = Ja
    engToUrk DaMeansNo Yes = Ja
    engToUrk DaMeansNo No = Da

    answerAs T eng = return $ engToUrk langMap eng
    answerAs F eng = return $ engToUrk langMap (flipEng eng)
    answerAs R eng = fmap ([Da, Ja] !!) $ randomRIO (0, 1)

    go (SDone p) = print (p == perm, p, perm)
    go (SQuestion to q whenDa whenJa) = do
      answer <- answerAs (perm !! to) (q (perm, perm !! to, langMap))
      case (answer :: Urk) of
        Da -> go whenDa
        Ja -> go whenJa

  go s

main :: IO ()
main = replicateM_ 10 $ sampleRun solution
```

[wikipedia]: https://en.wikipedia.org/wiki/The_Hardest_Logic_Puzzle_Ever
