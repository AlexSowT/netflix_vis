---
follows: dataRoot
---

```elm {l=hidden v}
boxPlotChart : Spec
boxPlotChart =
    let
        cfg =
            configure
                << configuration (coBackground netflix_darkGrey)
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coPadding (paEdges 60 30 30 40))

        trans =
            transform
                << VegaLite.filter (fiExpr "datum.type == 'Movie'")
                << calculateAs "slice( datum.duration, 0, -4)" "cleaned_duration"
                << calculateAs "split( datum.listed_in, ',')" "split_genres"
                << calculateAs "toNumber(substring(datum.show_id, 1))" "number_id"

        -- Box plot --
        encBox =
            encoding
                << position X
                    [ pName "release_year"
                    , pTemporal
                    , pAxis
                        [ axGrid False
                        , axTitle "Year of Release"
                        , axLabelColor netflix_white
                        , axTitleColor netflix_white
                        , axOffset 10
                        , axLabelOverlap osGreedy
                        , axDomain False
                        , axTitleFontSize 18
                        , axLabelFontSize 15
                        , axLabelExpr "'\\'' + substring(datum.label, 2)"
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
                        , axDomain False
                        , axTicks False
                        , axLabelPadding 10
                        , axTitleFontSize 18
                        , axLabelFontSize 15
                        ]
                    ]

        specBox =
            asSpec
                [ width 1400
                , height 400
                , padding (paSize 50)
                , title
                    "MOVIES ON NETFLIX"
                    [ tiFontSize 60
                    , tiColor netflix_red
                    , tiFont "./BebasNeue.otf"
                    , tiSubtitle "DURATION OF NETFLIX MOVIES THE YEAR THEY WERE RELEASED"
                    , tiSubtitleFont "./BebasNeue.otf"
                    , tiSubtitleFontSize 30
                    , tiSubtitleColor netflix_white
                    , tiSubtitlePadding 50
                    ]
                , encBox []
                , trans []
                , boxplot
                    [ maTicks
                        [ maColor netflix_white
                        , maSize 8
                        ]
                    , maBox [ maFill netflix_red ]
                    , maOutliers [ maColor netflix_white ]
                    , maRule [ maStroke netflix_white ]
                    ]
                ]

        --- End of boxplot ---
        -- Bar Chart of All Count --
        encBar =
            encoding
                << position X
                    [ pName "release_year"
                    , pTemporal
                    , pAxis
                        [ axGrid False
                        , axTitle "Year of Release"
                        , axLabelColor netflix_white
                        , axTitleColor netflix_white
                        , axOffset 10
                        , axDomain False
                        , axTitleFontSize 18
                        , axLabelFontSize 15
                        , axLabelExpr "'\\'' + substring(datum.label, 2)"
                        ]
                    ]
                << position Y
                    [ pAggregate opCount
                    , pTitle ""
                    , pAxis
                        [ axGridOpacity 0.1
                        , axTitle "Count of Movies"
                        , axLabelColor netflix_white
                        , axTitleColor netflix_white
                        , axDomain False
                        , axTicks False
                        , axLabelPadding 10
                        , axTitleFontSize 18
                        , axLabelFontSize 15
                        ]
                    ]

        specBar =
            asSpec
                [ width 1400
                , height 350
                , encBar []
                , trans []
                , bar [ maColor netflix_red ]
                , title
                    " "
                    [ tiFontSize 30
                    , tiColor netflix_red
                    , tiFont "./BebasNeue.otf"
                    , tiSubtitle "COUNT OF NETFLIX MOVIES"
                    , tiSubtitleFont "./BebasNeue.otf"
                    , tiSubtitleFontSize 30
                    , tiSubtitleColor netflix_white
                    , tiSubtitlePadding 30
                    ]
                ]

        --- End of ---
        encLineFacet =
            encoding
                << position X
                    [ pName "release_year"
                    , pTemporal
                    , pAxis
                        [ axGrid False
                        , axTitle "Year of Release"
                        , axLabelColor netflix_white
                        , axTitleColor netflix_white
                        , axDomain True
                        , axTitleFontSize 15
                        , axLabelOverlap osGreedy
                        , axLabelExpr "'\\'' + substring(datum.label, 2)"
                        , axLabelFontSize 15
                        ]
                    ]
                << position Y
                    [ pAggregate opCount
                    , pTitle ""
                    , pAxis
                        [ axGrid True
                        , axGridOpacity 0.1
                        , axTitle "Count of Movies"
                        , axLabelColor netflix_white
                        , axTitleColor netflix_white
                        , axTicks False
                        , axLabelPadding 10
                        , axTitleFontSize 13
                        , axValues (nums [ 0, 175, 350 ])
                        , axLabelFontSize 15
                        ]
                    ]

        -- Table of first 20 rows --
        transformFlatGenres =
            transform
                << calculateAs "toNumber(substring(datum.show_id, 1))" "number_id"
                << VegaLite.filter (fiExpr "datum.type == 'Movie'")
                << window [ ( [ wiOp woRank ], "rank" ) ] []
                << calculateAs "split( datum.listed_in, ',')" "split_genres"
                << flatten [ "split_genres" ]
                << calculateAs "trim(datum.split_genres)" "trimmed_genres"

        specFacet =
            asSpec
                [ columns 5
                , transformFlatGenres []
                , title
                    " "
                    [ tiFontSize 50
                    , tiColor netflix_red
                    , tiFont "./BebasNeue.otf"
                    , tiSubtitle ""
                    , tiSubtitleFont "./BebasNeue.otf"
                    , tiSubtitleFontSize 15
                    , tiSubtitleColor netflix_white
                    , tiSubtitlePadding 30
                    , tiAnchor anMiddle
                    ]
                , facetFlow
                    [ fName "trimmed_genres"
                    , fHeader
                        [ hdLabelColor netflix_white
                        , hdLabelFontSize 14
                        , hdTitle "COUNT OF MOVIES BY GENRE"
                        , hdTitleColor netflix_white
                        , hdTitleFontSize 30
                        , hdTitleFont "./BebasNeue.otf"
                        , hdTitleFontStyle "normal"
                        , hdTitlePadding 20
                        , hdTitleFontWeight fwNormal
                        ]
                    ]
                , specification
                    (asSpec
                        [ encLineFacet []
                        , line [ maStroke netflix_red ]
                        , height 60
                        , width 265
                        ]
                    )
                ]
    in
    toVegaLite
        [ data
        , cfg []
        , vConcat [ specBox, specBar, specFacet ]
        ]
```
