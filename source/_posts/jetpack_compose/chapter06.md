---
layout: post
title: "6장 조립"
categories: jetpack-compose-study
date: 2023-03-07
tags: [dev, android]
toc: true
---

[Source Code](https://github.com/Just-Android-Compose-Study/JetpackComposeStudy/tree/master/Chapter06)

## 컴포즈 앱 스타일링

Material You (Material 3) 사용하면서 변경사항이 좀 많은 것 같아 레퍼런스 참고
> - https://m3.material.io/
> - [Compose의 Material 2에서 Material 3으로 이전](https://developer.android.com/jetpack/compose/designsystems/material2-material3?hl=ko)

### 색상, 모양, 텍스트 스타일 정의

예제코드 중 Material3이 반영되지 않은 코드는 변환해가며 작성해봤다.

부모 테마를 재정의해 테마를 중첩할 수 있다.

```kotlin
@Composable
@Preview
fun MaterialThemeDemo() {
    MaterialTheme(
        typography = Typography.copy(	// Typography 재사용
            displayLarge = TextStyle(color = Color.Red)
        )
    ) {
        Row(
            Modifier.fillMaxSize(),
            verticalAlignment = Alignment.CenterVertically,
            horizontalArrangement = Arrangement.Center
        ) {
            Text(
                text = "Hello",
                style = MaterialTheme.typography.displayLarge	// 수정된 MaterialTheme
            )
            Spacer(modifier = Modifier.width(2.dp))
            MaterialTheme(
                typography = Typography.copy(	// Typography 재사용
                    displayLarge = TextStyle(color = Color.Blue)
                )
            ) {
                Text(
                    text = "Compose",
                    style = MaterialTheme.typography.displayLarge // 수정된 MaterialTheme
                )
            }
        }
    }
}
```

### 리소스 기반의 테마 사용

테마 xml에도 컬러 값을 사용하고 컴포즈에서도 컬러 값을 또 선언하지 않으려면 ```colorResource()```을 사용해 리소스 쪽에서만 컬러 값을 정리해두자.

```kotlin
val colorScheme = when {
    dynamicColor && Build.VERSION.SDK_INT >= Build.VERSION_CODES.S -> {
        val context = LocalContext.current
        // Android 31 미만에서는 다크모드 X
        if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
    }
    darkTheme -> DarkColorScheme
    else -> LightColorScheme.copy(secondary = colorResource(id = R.color.orange_dark))
}
```

SplashScreen을 안드로이드 12 이전에서도 사용할 수 있는 방법도 있다니 찾아보자.

#### values/themes.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <style name="Theme.AnimatedSplashScreen" parent="android:Theme.Material.Light.NoActionBar">
        <item name="android:statusBarColor">@color/black</item>
        <item name="android:windowBackground">@color/black</item>
    </style>
</resources>
```

#### values-31/themes.xml

<?xml version="1.0" encoding="utf-8"?>
<resources>

    <style name="Theme.AnimatedSplashScreen" parent="android:Theme.Material.Light.NoActionBar">
        <item name="android:statusBarColor">@color/black</item>
        <item name="android:windowBackground">@color/black</item>
        <item name="android:windowSplashScreenAnimatedIcon">@drawable/transparent_image</item>
    </style>
</resources>

- splash icon을 투명하게 만들고 SplashScreen이 제일 처음에 보이도록 한다. (이렇게 안하면 SplashScreen 앞에 앱 아이콘이 잠시 보임)

## 툴바와 메뉴 통합

- Scaffold()에서 topBar, bottomBar 정의

```kotlin
@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun ComposeUnitConverter(factory: ViewModelFactory) {
    val navController = rememberNavController()	// 화면을 이동하기 위한 NavHostController 생성
    val menuItems = listOf("Item #1", "Item #2")

    // Material3: ScaffoldState 사용 안하고 바로 snackbarHostState 선언
    val snackbarHostState = remember { SnackbarHostState() }
    val snackbarCoroutineScope = rememberCoroutineScope()

    Chapter06Theme(dynamicColor = false) {
        Scaffold(topBar = {
            ComposeUnitConverterTopBar(menuItems) { s ->
                snackbarCoroutineScope.launch {
                    snackbarHostState.showSnackbar(s)
                }
            }
        }, bottomBar = {
            ComposeUnitConverterBottomBar(navController)
        }) {
            ComposeUnitConverterNavHost(
                navController = navController, factory = factory, modifier = Modifier.padding(it)
            )
        }
    }
}
```

### 상단 앱 바 생성

- TopAppBar() 사용하면 된다.

```kotlin
@ExperimentalMaterial3Api
@Composable
fun TopAppBar(
    title: @Composable () -> Unit,
    modifier: Modifier = Modifier,
    navigationIcon: @Composable () -> Unit = {},	// 앱 바 좌측 아이콘
    actions: @Composable RowScope.() -> Unit = {},	// 앱 바 우측에 들어가는 Row() 
    windowInsets: WindowInsets = TopAppBarDefaults.windowInsets,
    colors: TopAppBarColors = TopAppBarDefaults.smallTopAppBarColors(),
    scrollBehavior: TopAppBarScrollBehavior? = null	// 스크롤 상태에 따라 투명도를 조절하는 행동을 넣을 수 있겠다...
) {
    SingleRowTopAppBar(
        modifier = modifier,
        title = title,
        titleTextStyle = MaterialTheme.typography.fromToken(TopAppBarSmallTokens.HeadlineFont),
        centeredTitle = false,
        navigationIcon = navigationIcon,
        actions = actions,
        windowInsets = windowInsets,
        colors = colors,
        scrollBehavior = scrollBehavior
    )
}
```

예제 코드에는 DropdownMenu까지 action에 추가해서 보여주고 있지만 여기서는 중요하진 않아서 기록하지는 않겠음.


## 네비게이션 추가

Scaffold()에서 bottomBar로 추가한 BottomAppBar() 알아보기

화면별로 라우팅을 하기 위한 객체가 하나 필요하다.

```kotlin
sealed class ComposeUnitConverterScreen(
    val route: String,
    @StringRes val label: Int,
    @DrawableRes val icon: Int
) {
    companion object {
        val screens = listOf(
            Temperature,
            Distances
        )

        const val route_temperature = "temperature"
        const val route_distances = "distances"
    }

    private object Temperature : ComposeUnitConverterScreen(
        route_temperature,
        R.string.temperature,
        R.drawable.baseline_thermostat_24
    )

    private object Distances : ComposeUnitConverterScreen(
        route_distances,
        R.string.distances,
        R.drawable.baseline_square_foot_24
    )
}
```

- 이제 이 ComposeUnitConverterScreen 안에 있는 screens 리스트에서 원하는 화면을 고르면 그 화면의 컴포저블 함수를 리턴해준다.
- 아래는 ```ComposeUnitConverterScreen```를 어떻게 사용하는지 나온다.

```kotlin
@Composable
fun ComposeUnitConverterBottomBar(navController: NavHostController) {
    NavigationBar {
        val navBackStackEntry by navController.currentBackStackEntryAsState()
        val currentDestination = navBackStackEntry?.destination
        // 모든 스크린 forEach
        ComposeUnitConverterScreen.screens.forEach { screen ->
            NavigationBarItem(selected = currentDestination?.hierarchy?.any { it.route == screen.route } == true,
                onClick = {
                    // 일단 클릭하면 그 화면으로 이동하기
                    navController.navigate(screen.route) {
                        launchSingleTop = true
                    }
                },
                label = {
                    // screen 객체에 이미 정의된 label
                    Text(text = stringResource(id = screen.label))
                },
                icon = {
                    // screen 객체에 이미 정의된 icon, label
                    Icon(
                        painter = painterResource(id = screen.icon),
                        contentDescription = stringResource(id = screen.label)
                    )
                },
                alwaysShowLabel = false	// 선택되었을 때만 label 보여줌
            )
        }
    }
}
```

### NavHostController와 NavHost() 사용

교재는 **섭씨/화씨 변환 기능이 있는 화면**과 **미터/마일 변환 기능이 있는 화면**을 ```NavHostController```와 ```NavHost```를 이용해 전환하는 예제를 보여주고 있다.

```kotlin
@Composable
fun ComposeUnitConverterNavHost(
    navController: NavHostController, factory: ViewModelProvider.Factory?, modifier: Modifier
) {
    NavHost(
        navController = navController,
        startDestination = ComposeUnitConverterScreen.route_temperature,
        modifier = modifier
    ) {
        composable(ComposeUnitConverterScreen.route_temperature) {
            TemperatureConverter(
                viewModel = viewModel(factory = factory)
            )
        }
        composable(ComposeUnitConverterScreen.route_distances) {
            DistancesConverter(
                viewModel = viewModel(factory = factory)
            )
        }
    }
}
```

근데 아까 위에서 Splash 화면에서 홈화면으로 이동하기 위해서도 이러한 동작이 필요했었다.

MainActivity부터 살펴보자

```kotlin
class MainActivity: ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            Chapter06Theme {
                val navController = rememberNavController()
                SetupNavGraph(navController = navController)
            }
        }
    }
}
```

이 Activity에서 이동할 화면들은 모두 NavHostController로 제어한다.

```kotlin
@Composable
fun SetupNavGraph(navController: NavHostController) {
    NavHost(navController = navController, startDestination = Screen.Splash.route) {
        composable(route = Screen.Splash.route) {
            SplashScreen(navController = navController)
        }
        composable(route = Screen.Home.route) {
            HomeScreen()
        }
    }
}
```

여기서는 화면이동 단위를 SplashScreen과 HomeScreen으로 나눴다.
이 HomeScreen 안에서는 아까 위에서 섭씨/화씨, 미터/마일 변환하는 화면이 나올 것이다.
그 화면들을 여기서 전부 통합해 정의하면 어떻게 될까...?


```kotlin
@Composable
fun SetupNavGraph(navController: NavHostController) {
    NavHost(navController = navController, startDestination = ComposeUnitConverterScreen.splash) {
        composable(ComposeUnitConverterScreen.splash) {
            SplashScreen(navController = navController)
        }
        composable(ComposeUnitConverterScreen.temperature) {
            HomeScreen(navController = navController)
        }
        composable(ComposeUnitConverterScreen.distances) {
            HomeScreen(navController = navController)
        }
    }
}
```

이렇게 구성했더니 HomeScreen에서 화면을 이동할 때마다 깜빡거린다. 확실히 이건 잘못된 방법이었다. ㅠㅠ

이 예제는 Navigation 기능을 사용해 화면을 이동하는 예제다 보니 화면 전환했던 기록이 전부 쌓여있어서 Back 버튼을 누르면 이전 화면으로 되돌아가는게 보인다. (데이터도 이전 화면의 데이터가 남아있음)

그리고 ViewModelFactory를 사용하다보니 화면을 새로 만들면서도 이전 값이 유지 되면서 생성된다는 점도 같이 확인하면 되겠다.
