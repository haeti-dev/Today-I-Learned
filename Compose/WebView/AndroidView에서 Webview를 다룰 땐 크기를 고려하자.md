## 문제 상황

```kotlin
    AndroidView(
        factory = { context ->
            WebView(context).apply {
                with(settings) {
                    javaScriptEnabled = true
                    loadWithOverviewMode = true
                    useWideViewPort = true
                    supportMultipleWindows()
                    setSupportZoom(false)
                    javaScriptCanOpenWindowsAutomatically = true
                    loadsImagesAutomatically = true
                    mixedContentMode = WebSettings.MIXED_CONTENT_ALWAYS_ALLOW
                    userAgentString += "android"
                }
                webViewClient = WebViewClient()
                webChromeClient = object : WebChromeClient() {
                    override fun onConsoleMessage(consoleMessage: ConsoleMessage): Boolean {
                        Log.e("ComposeWebView", "onConsoleMessage: ${consoleMessage.message()}")
                        return super.onConsoleMessage(consoleMessage)
                    }
                }
                webView = this
            }
        },
        update = { view ->
            view.loadUrl(url)
        },
        onRelease = {
            (it as? WebView)?.destroy()
       
        modifier = Modifier.fillMaxSize(),
    )
```

위와 같이 `Modifier.fillMaxSize()`를 적용했음에도, 웹뷰 상단 부분 여백이 생략되고, 웹뷰에서 중간 부분에 위치한 컴포넌트가 화면의 최상단에 보였다. 

하지만 xml 웹뷰에서 동일한 settings, client를 설정하니 웹뷰가 짤리지 않고 잘 보였다. 따라서 컴포즈에서 설정 문제인 것을 확인하였다.

## 해결 방법

Compose의 AndroidView가 부모가 제공하는 최대 공간인 Modifier.fillMaxSize()를 할당받더라도, factory에서 생성한 View의 기본 layoutParams가 WRAP_CONTENT로 동작한다면, 내부 WebView가 원하는 크기(실제 웹 콘텐츠 크기)에 맞춰져서 그려질 수 있다. 

따라서 아래와 같이 layoutParams를 MATCH_PARENT로 직접 설정하면, WebView가 Compose가 준 사이즈 그대로 레이아웃될 수 있다. 

```kotlin
 AndroidView(
        factory = { context ->
            WebView(context).apply {
                layoutParams = ViewGroup.LayoutParams(
                    ViewGroup.LayoutParams.MATCH_PARENT,
                    ViewGroup.LayoutParams.MATCH_PARENT,
                )
                with(settings) {
                    javaScriptEnabled = true
                    loadWithOverviewMode = true
                    useWideViewPort = true
                    supportMultipleWindows()
                    setSupportZoom(false)
                    javaScriptCanOpenWindowsAutomatically = true
                    loadsImagesAutomatically = true
                    mixedContentMode = WebSettings.MIXED_CONTENT_ALWAYS_ALLOW
                    userAgentString += "android"
                }
                webViewClient = WebViewClient()
                webChromeClient = object : WebChromeClient() {
                    override fun onConsoleMessage(consoleMessage: ConsoleMessage): Boolean {
                        Log.e("ComposeWebView", "onConsoleMessage: ${consoleMessage.message()}")
                        return super.onConsoleMessage(consoleMessage)
                    }
                }
                webView = this
            }
        },
        update = { view ->
            view.loadUrl(url)
        },
        onRelease = {
            (it as? WebView)?.destroy()
        },
        modifier = Modifier.fillMaxSize(),
    )
```

Compose가 충분한 크기를 제공해도, View가 “나는 내 콘텐츠 크기에 맞춰 그려질래”라고 응답한다면 실제 배치 결과가 원하는 크기보다 작게 그려질 수 있다. 

일반적으로는 부모가 전달한 제약에 맞춰서 웹뷰가 스스로의 크기를 계산하는데, 보통 실제 크기만큼만 레이아웃이 잡혀서 결과적으로 WRAP_CONTENT와 유사하게 동작한다. 따라서 전체 화면을 다 웹뷰로 채우고 싶은 경우 또는 반응형 웹/스크롤이 필요한 경우 MATCH_PARENT를 사용하는 것이 적합하다.
