```kotlin
        /**
         * Creates a vertical gradient with the given colors at the provided offset defined
         * in the [Pair<Float, Color>]
         *
         * Ex:
         * ```
         *  Brush.verticalGradient(
         *      0.1f to Color.Red,
         *      0.3f to Color.Green,
         *      0.5f to Color.Blue,
         *      startY = 0.0f,
         *      endY = 100.0f
         * )
         * ```
         *
         * @sample androidx.compose.ui.graphics.samples.VerticalGradientColorStopSample
         * @sample androidx.compose.ui.graphics.samples.GradientBrushSample
         *
         * @param colorStops Colors and offsets to determine how the colors are dispersed throughout
         * the vertical gradient
         * @param startY Starting y position of the vertical gradient. Defaults to 0 which
         * represents the top of the drawing area
         * @param endY Ending y position of the vertical gradient.
         * Defaults to [Float.POSITIVE_INFINITY] which indicates the bottom of the specified
         * drawing area
         * @param tileMode Determines the behavior for how the shader is to fill a region outside
         * its bounds. Defaults to [TileMode.Clamp] to repeat the edge pixels
         */
        @Stable
        fun verticalGradient(
            vararg colorStops: Pair<Float, Color>,
            startY: Float = 0f,
            endY: Float = Float.POSITIVE_INFINITY,
            tileMode: TileMode = TileMode.Clamp
        ): Brush = linearGradient(
            *colorStops,
            start = Offset(0.0f, startY),
            end = Offset(0.0f, endY),
            tileMode = tileMode
        )

```

<br/>

Compose에서는 Modifier.background에서 Brush.verticalGradient로 세로 방향 그라디언트 효과를 줄 수 있다. 각 속성에 대한 설명은 아래와 같다. 

- `colorStops`: (Float to Color) 형태로, 어떤 지점(offset)에서 어떤 색상을 적용할지를 지정
- `startY`: 세로 그라디언트가 시작되는 위치(px 단위).
- `endY`: 세로 그라디언트가 끝나는 위치(px 단위).
- `tileMode`: 그라디언트 범위를 벗어났을 때 색상이 어떻게 채워질지를 결정한다. 일반적으로TileMode.Clamp를 사용하면 그 범위를 넘어도 마지막 색상이 계속 채워진다.

<br/>

colorStops는 그라디언트를 아주 섬세하게 다룰 수 있는 속성이다.

<br/>

```kotlin
Brush.verticalGradient(
    0.0f to Color.Red,    // offset=0.0 -> 그라디언트 시작 부분에서 Color.Red
    0.5f to Color.Green,  // offset=0.5 -> 그라디언트 중앙쯤에서 Color.Green
    1.0f to Color.Blue    // offset=1.0 -> 그라디언트 끝 부분에서 Color.Blue
)
```

<br/>

위 예시에서 알 수 있듯이, 어떤 지점에서 어떤 색으로 보여지게 할 건지 `Pair<Float, Color>` 형태로 정의할 수 있다. 

Brush.linearGradient, Brush.horizontalGradient, Brush.radialGradient 등도 같은 방식으로 colorStops를 받는다. 단, **방향**이나 **형태**가 다를 뿐, 색상과 오프셋을 정의하는 원리는 동일하다.

타일모드도 offset 범위 밖에서도 패턴처럼 반복되거나 반사된 형태로 보여지게 할 때 유용하게 사용될 것 같다.
