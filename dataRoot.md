---
id: litvis

elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

```elm {l=hidden}
import String exposing (..)
import VegaLite exposing (..)
```

Importing the data from the csv

```elm {l}
data : Data
data =
    dataFromUrl "./netflix_titles.csv" []
```

```elm {l=hidden}
netflix_red : String
netflix_red =
    "#E50914"


netflix_white : String
netflix_white =
    "#F5F5F1"


netflix_darkGrey : String
netflix_darkGrey =
    "#221F1F"
```

```elm {l r}
fromJustInt : Maybe Int -> Int
fromJustInt x =
    case x of
        Just y ->
            y

        Nothing ->
            -1
```

```elm {l r}
test : Int
test =
    fromJustInt (toInt (dropLeft 1 "M23"))
```
