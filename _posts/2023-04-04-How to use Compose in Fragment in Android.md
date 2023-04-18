---
layout: post
title: How to use Compose in XML layout in Android
subtitle: Compose in XML
published: true
categories: Android
tags: [Android, Compose]
---

View기반 화면(XML)에서 Compose를 사용하는 방법
=============  

>Compose는 상호 운용이 가능하여 뷰 기반(XML) 화면 에서도 사용할 수 있다.


### 1. Compose화면 코드 제작  

Compose를 사용하기 위해서는 Composable 함수를 만들어야 합니다.

```kotlin
MyComposable.kt  
  
@Composable
fun MyComposable() {
    Scaffold(
        topBar = {
            TopAppBar(
                title = {  Text(text = "Compose in Fragment")  }
            )
        },
        content = {
            Column(
                modifier = Modifier.padding(16 dp),
                verticalArrangement = Arrangement.Center,
                horizontalAlignment = Alignment.CenterHorizontally
            ) {
                TextField(
                    value = "이름 입력",
                    onValueChange = { /* 이름 입력 시 로직 */  },
                    label = { Text("이름") }
                )
            }
        }
    )
}
```  

### 2. View기반 화면 제작  
  
### __Fragment를 사용하는 경우__
MyComposable을 사용할 Fragment를 제작합니다.  


```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
	>

    // ComposeView를 추가합니다.
    <androidx.compose.ui.platform.ComposeView
        android:id="@+id/compose_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        />

</androidx.constraintlayout.widget.ConstraintLayout>  
```  

```kotlin
class MyComposeFragment : Fragment() {
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        binding = FragmentMyComposeBinding.inflate(inflater, container, false)
        binding.composeView.apply {
            setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnLifecycleDestroyed)
            setContent {
                // 여기에 Composable을 넣습니다.
                MyComposable()
            }
        }

        return binding.root
    }
}
```  

### __Activity를 사용하는 경우__
MyComposable을 사용할 Activity를 제작합니다.  

```kotlin
class MyComposeActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMyComposeBinding.inflate(layoutInflater)
        binding.composeView.apply {
            setViewCompositionStrategy(ViewCompositionStrategy.DisposeOnLifecycleDestroyed)
            setContent {
                // 여기에 Composable을 넣습니다.
                MyComposable()
            }
        }
    }
}
```  

__Activity class__ 를 제작할 때, 보통 __ComponentActivity__ 또는 __AppCompatActivity를__ 상속받아 사용하는데, 
__Android__ 하위 버전 호환 없이, 최신 버전만 지원하고자 하거나, __Compose__ 만으로 __App__ 을 제작하려고 하면 __ComponentActivity__ 를 상속받아 사용하면 됩니다.

__보통은 AppCompatActivity를 사용하면 됩니다.__

**setContent에 보여줄 화면 함수를 넣어주면 됩니다.**  

### 화면이 나오고, Compose를 사용할 수 있습니다.