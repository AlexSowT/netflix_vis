---
follows: dataRoot
---

```elm {v}
dotChart : Spec
dotChart =
    let
        cfg =
            configure
                << configuration (coBackground netflix_darkGrey)
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coPadding (paEdges 20 30 20 30))

        enc =
            encoding
                << position X
                    [ pName "release_year"
                    , pAxis
                        [ axGrid False
                        , axTitle "Year of Release"
                        , axLabelColor netflix_white
                        , axTitleColor netflix_white
                        , axLabelAngle 40
                        , axOffset 10
                        ]
                    ]
                << position Y
                    [ pName "cleaned_duration"
                    , pQuant
                    , pTitle ""
                    , pAxis
                        [ axGridOpacity 0.1
                        , axTitle "Duration In Minutes"
                        , axLabelColor netflix_white
                        , axTitleColor netflix_white
                        ]
                    ]

        trans =
            transform
                << VegaLite.filter (fiExpr "datum.type == 'Movie'")
                << calculateAs "slice( datum.duration, 0, -4)" "cleaned_duration"

        --                << aggregate [ opAs opCount "scaledDate" "aggregated" ] [ "decade" ]
    in
    toVegaLite
        [ width 1500
        , height 500
        , data
        , cfg []
        , enc []
        , trans []
        , point [ maOpacity 0.7, maColor netflix_red ]
        ]
```
