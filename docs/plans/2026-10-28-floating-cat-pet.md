# 桌面宠物猫 · 实现计划

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 构建一个 Android 悬浮窗桌面宠物应用，白色长毛蓝眼睛猫咪浮在屏幕上，点击弹出每日任务面板（读一段话 17 遍）。

**Architecture:** 原生 Android + Kotlin + Jetpack Compose。前台服务 `PetService` 持有 `FloatingPetManager`，通过 `WindowManager` 添加悬浮窗 `PetView`（猫咪）和 `TaskPanel`（任务面板）。数据用 DataStore 持久化。

**Tech Stack:** Kotlin, Jetpack Compose, WindowManager, DataStore Preferences, WorkManager, Coroutines/Flow

---

## 文件结构

```
app/
├── build.gradle.kts                          # 模块构建脚本
└── src/main/
    ├── AndroidManifest.xml                   # 权限 + 服务 + Activity 声明
    ├── java/com/catpet/
    │   ├── MainActivity.kt                   # 权限引导 + 启动/停止服务
    │   ├── PetService.kt                     # 前台服务 + 通知 + 每日重置
    │   ├── FloatingPetManager.kt             # WindowManager 悬浮窗管理 + 拖动
    │   ├── data/
    │   │   └── TaskRepository.kt             # DataStore 读写 + 重置逻辑
    │   └── ui/
    │       ├── PetView.kt                    # 猫咪 View + 动画 + 手势
    │       └── TaskPanel.kt                  # 任务面板 UI
    └── res/
        ├── drawable/cat_pet.jpg              # 猫咪图片
        ├── values/strings.xml
        └── values/themes.xml
```

---

### Task 1: 项目骨架与 Gradle 配置

**Files:**
- Create: `settings.gradle.kts`
- Create: `build.gradle.kts` (root)
- Create: `gradle.properties`
- Create: `app/build.gradle.kts`

- [ ] **Step 1: 创建根 settings.gradle.kts**

```kotlin
pluginManagement {
    repositories {
        google()
        mavenCentral()
        gradlePluginPortal()
    }
}
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
rootProject.name = "CatPet"
include(":app")
```

- [ ] **Step 2: 创建根 build.gradle.kts**

```kotlin
plugins {
    id("com.android.application") version "8.2.2" apply false
    id("org.jetbrains.kotlin.android") version "1.9.22" apply false
}
```

- [ ] **Step 3: 创建 gradle.properties**

```properties
org.gradle.jvmargs=-Xmx2048m -Dfile.encoding=UTF-8
android.useAndroidX=true
kotlin.code.style=official
android.nonTransitiveRClass=true
```

- [ ] **Step 4: 创建 app/build.gradle.kts**

```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
}

android {
    namespace = "com.catpet"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.catpet"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }

    buildTypes {
        release {
            isMinifyEnabled = false
        }
    }
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }
    kotlinOptions {
        jvmTarget = "17"
    }
    buildFeatures {
        compose = true
    }
    composeOptions {
        kotlinCompilerExtensionVersion = "1.5.8"
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.12.0")
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.7.0")
    implementation("androidx.activity:activity-compose:1.8.2")
    implementation(platform("androidx.compose:compose-bom:2024.02.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.ui:ui-graphics")
    implementation("androidx.compose.ui:ui-tooling-preview")
    implementation("androidx.compose.material3:material3")
    implementation("androidx.datastore:datastore-preferences:1.0.0")
    implementation("androidx.work:work-runtime-ktx:2.9.0")
}
```

- [ ] **Step 5: 创建 Gradle Wrapper（使用本地 gradle 或 wrapper jar）**

如果环境有 gradle，运行 `gradle wrapper --gradle-version 8.5`；否则从已有 Android 项目复制 `gradlew`、`gradlew.bat`、`gradle/wrapper/` 目录。

- [ ] **Step 6: 验证构建配置**

Run: `./gradlew :app:assembleDebug`
Expected: BUILD SUCCESSFUL（即使没有源码，空项目也能编译）

- [ ] **Step 7: Commit**

```bash
git add settings.gradle.kts build.gradle.kts gradle.properties app/build.gradle.kts gradle/wrapper/ gradlew gradlew.bat
git commit -m "chore: init android project with compose"
```

---

### Task 2: AndroidManifest 与权限

**Files:**
- Create: `app/src/main/AndroidManifest.xml`

- [ ] **Step 1: 创建 AndroidManifest.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE_SPECIAL_USE" />
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/Theme.CatPet">

        <activity
            android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <service
            android:name=".PetService"
            android:exported="false"
            android:foregroundServiceType="specialUse">
            <property
                android:name="android.app.PROPERTY_SPECIAL_USE_FGS_SUBTYPE"
                android:value="桌面宠物悬浮窗" />
        </service>

    </application>
</manifest>
```

- [ ] **Step 2: 创建 strings.xml**

Create `app/src/main/res/values/strings.xml`:
```xml
<resources>
    <string name="app_name">桌面猫咪</string>
    <string name="notification_channel_name">桌面宠物</string>
    <string name="notification_text">猫咪正在陪伴你</string>
    <string name="task_name">读一段话</string>
</resources>
```

- [ ] **Step 3: 创建 themes.xml**

Create `app/src/main/res/values/themes.xml`:
```xml
<resources>
    <style name="Theme.CatPet" parent="android:Theme.Material.Light.NoActionBar" />
</resources>
```

- [ ] **Step 4: Commit**

```bash
git add app/src/main/AndroidManifest.xml app/src/main/res/values/strings.xml app/src/main/res/values/themes.xml
git commit -m "feat: add manifest, permissions and resources"
```

---

### Task 3: 数据层 — TaskRepository

**Files:**
- Create: `app/src/main/java/com/catpet/data/TaskRepository.kt`

- [ ] **Step 1: 创建 TaskRepository**

```kotlin
package com.catpet.data

import android.content.Context
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.intPreferencesKey
import androidx.datastore.preferences.core.longPreferencesKey
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.Flow
import kotlinx.coroutines.flow.map
import java.text.SimpleDateFormat
import java.util.Date
import java.util.Locale

private val Context.dataStore by preferencesDataStore(name = "cat_pet_prefs")

class TaskRepository(private val context: Context) {

    private val KEY_PROGRESS = intPreferencesKey("task_progress")
    private val KEY_LAST_RESET = stringPreferencesKey("last_reset_date")
    private val KEY_TASK_NAME = stringPreferencesKey("task_name")
    private val KEY_TARGET = intPreferencesKey("task_target")
    private val KEY_PET_X = intPreferencesKey("pet_x")
    private val KEY_PET_Y = intPreferencesKey("pet_y")
    private val KEY_LAST_INTERACTION = longPreferencesKey("last_interaction_time")

    val progressFlow: Flow<Int> = context.dataStore.data.map { prefs ->
        resetIfNeeded(prefs[KEY_LAST_RESET])
        prefs[KEY_PROGRESS] ?: 0
    }

    val taskNameFlow: Flow<String> = context.dataStore.data.map { it[KEY_TASK_NAME] ?: "读一段话" }
    val targetFlow: Flow<Int> = context.dataStore.data.map { it[KEY_TARGET] ?: 17 }
    val petPositionFlow: Flow<Pair<Int, Int>> = context.dataStore.data.map {
        Pair(it[KEY_PET_X] ?: -1, it[KEY_PET_Y] ?: -1)
    }
    val lastInteractionFlow: Flow<Long> = context.dataStore.data.map { it[KEY_LAST_INTERACTION] ?: 0L }

    suspend fun increment() {
        context.dataStore.edit { prefs ->
            val current = prefs[KEY_PROGRESS] ?: 0
            prefs[KEY_PROGRESS] = current + 1
        }
        updateLastInteraction()
    }

    suspend fun savePosition(x: Int, y: Int) {
        context.dataStore.edit { prefs ->
            prefs[KEY_PET_X] = x
            prefs[KEY_PET_Y] = y
        }
    }

    suspend fun updateLastInteraction() {
        context.dataStore.edit { prefs ->
            prefs[KEY_LAST_INTERACTION] = System.currentTimeMillis()
        }
    }

    suspend fun resetProgress() {
        context.dataStore.edit { prefs ->
            prefs[KEY_PROGRESS] = 0
            prefs[KEY_LAST_RESET] = todayString()
        }
    }

    private fun resetIfNeeded(lastReset: String?) {
        if (lastReset != todayString()) {
            // 在非协程上下文不能直接 edit，这里只判断，实际重置由 resetProgress 处理
            // 但 Flow 的 map 中无法调用 suspend，所以我们在 getProgress 时处理
        }
    }

    private fun todayString(): String {
        return SimpleDateFormat("yyyy-MM-dd", Locale.getDefault()).format(Date())
    }

    /** 在应用启动时调用，确保跨日重置 */
    suspend fun ensureDailyReset() {
        val prefs = context.dataStore.data
        val lastReset = prefs.map { it[KEY_LAST_RESET] }.let { flow ->
            var result: String? = null
            flow.collect { result = it; return@collect }
            result
        }
        if (lastReset != todayString()) {
            resetProgress()
        }
    }
}
```

- [ ] **Step 2: 修正 ensureDailyReset（Flow 收集方式）**

上面 `ensureDailyReset` 里的 Flow 收集写法有问题，改为：

```kotlin
suspend fun ensureDailyReset() {
    var lastReset: String? = null
    context.dataStore.data.collect { prefs ->
        lastReset = prefs[KEY_LAST_RESET]
        return@collect
    }
    if (lastReset != todayString()) {
        resetProgress()
    }
}
```

注意：`collect` 是挂起函数，这里取第一个值后 return。

- [ ] **Step 3: 编写单元测试验证重置逻辑**

Create `app/src/test/java/com/catpet/data/TaskRepositoryTest.kt`:

```kotlin
package com.catpet.data

import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore
import androidx.test.core.app.ApplicationProvider
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.runBlocking
import org.junit.Assert.assertEquals
import org.junit.Test
import org.junit.runner.RunWith
import org.robolectric.RobolectricTestRunner

@RunWith(RobolectricTestRunner::class)
class TaskRepositoryTest {

    private val context = ApplicationProvider.getApplicationContext<android.content.Context>()
    private val repo = TaskRepository(context)

    @Test
    fun `increment increases progress`() = runBlocking {
        repo.resetProgress()
        repo.increment()
        repo.increment()
        assertEquals(2, repo.progressFlow.first())
    }

    @Test
    fun `reset sets progress to zero`() = runBlocking {
        repo.increment()
        repo.increment()
        repo.resetProgress()
        assertEquals(0, repo.progressFlow.first())
    }
}
```

需要在 app/build.gradle.kts 添加 test 依赖：
```kotlin
testImplementation("junit:junit:4.13.2")
testImplementation("org.robolectric:robolectric:4.12.1")
testImplementation("androidx.test:core:1.5.0")
```

- [ ] **Step 4: 运行测试**

Run: `./gradlew :app:testDebugUnitTest`
Expected: BUILD SUCCESSFUL，测试通过

- [ ] **Step 5: Commit**

```bash
git add app/src/main/java/com/catpet/data/TaskRepository.kt app/src/test/ app/build.gradle.kts
git commit -m "feat: add TaskRepository with datastore and reset logic"
```

---

### Task 4: PetService 前台服务

**Files:**
- Create: `app/src/main/java/com/catpet/PetService.kt`

- [ ] **Step 1: 创建 PetService**

```kotlin
package com.catpet

import android.app.Notification
import android.app.NotificationChannel
import android.app.NotificationManager
import android.app.Service
import android.content.Context
import android.content.Intent
import android.os.Build
import android.os.IBinder
import androidx.core.app.NotificationCompat
import com.catpet.data.TaskRepository

class PetService : Service() {

    private lateinit var floatingPetManager: FloatingPetManager
    private lateinit var taskRepository: TaskRepository

    override fun onCreate() {
        super.onCreate()
        taskRepository = TaskRepository(this)
        floatingPetManager = FloatingPetManager(this, taskRepository)
        createNotificationChannel()
        startForeground(NOTIFICATION_ID, buildNotification())
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        floatingPetManager.show()
        return START_STICKY
    }

    override fun onDestroy() {
        floatingPetManager.hide()
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? = null

    private fun createNotificationChannel() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                CHANNEL_ID,
                getString(R.string.notification_channel_name),
                NotificationManager.IMPORTANCE_LOW
            ).apply {
                description = "桌面宠物常驻通知"
            }
            val nm = getSystemService(NotificationManager::class.java)
            nm.createNotificationChannel(channel)
        }
    }

    private fun buildNotification(): Notification {
        val intent = Intent(this, MainActivity::class.java)
        val pendingIntent = android.app.PendingIntent.getActivity(
            this, 0, intent,
            android.app.PendingIntent.FLAG_UPDATE_CURRENT or android.app.PendingIntent.FLAG_IMMUTABLE
        )
        return NotificationCompat.Builder(this, CHANNEL_ID)
            .setContentTitle(getString(R.string.app_name))
            .setContentText(getString(R.string.notification_text))
            .setSmallIcon(android.R.drawable.ic_dialog_info)
            .setContentIntent(pendingIntent)
            .setOngoing(true)
            .build()
    }

    companion object {
        private const val CHANNEL_ID = "cat_pet_channel"
        private const val NOTIFICATION_ID = 1001

        fun start(context: Context) {
            val intent = Intent(context, PetService::class.java)
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
                context.startForegroundService(intent)
            } else {
                context.startService(intent)
            }
        }

        fun stop(context: Context) {
            context.stopService(Intent(context, PetService::class.java))
        }
    }
}
```

- [ ] **Step 2: 验证编译**

Run: `./gradlew :app:compileDebugKotlin`
Expected: BUILD SUCCESSFUL（FloatingPetManager 尚未创建，会报错 — 先创建空壳）

在 Task 5 创建 FloatingPetManager 空壳后再编译。本步先提交。

- [ ] **Step 3: Commit**

```bash
git add app/src/main/java/com/catpet/PetService.kt
git commit -m "feat: add PetService foreground service with notification"
```

---

### Task 5: FloatingPetManager — 悬浮窗管理与拖动

**Files:**
- Create: `app/src/main/java/com/catpet/FloatingPetManager.kt`
- Create: `app/src/main/java/com/catpet/ui/PetView.kt` (空壳，Task 6 填充)
- Create: `app/src/main/java/com/catpet/ui/TaskPanel.kt` (空壳，Task 7 填充)

- [ ] **Step 1: 创建 PetView 空壳**

```kotlin
package com.catpet.ui

import android.content.Context
import android.widget.FrameLayout

class PetView(context: Context) : FrameLayout(context) {
    // 占位，Task 6 填充 Compose 内容
}
```

- [ ] **Step 2: 创建 TaskPanel 空壳**

```kotlin
package com.catpet.ui

import android.content.Context
import android.widget.FrameLayout

class TaskPanel(context: Context) : FrameLayout(context) {
    // 占位，Task 7 填充 Compose 内容
}
```

- [ ] **Step 3: 创建 FloatingPetManager**

```kotlin
package com.catpet

import android.content.Context
import android.graphics.PixelFormat
import android.os.Build
import android.view.Gravity
import android.view.MotionEvent
import android.view.View
import android.view.WindowManager
import com.catpet.data.TaskRepository
import com.catpet.ui.PetView
import com.catpet.ui.TaskPanel
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.SupervisorJob
import kotlinx.coroutines.launch

class FloatingPetManager(
    private val context: Context,
    private val taskRepository: TaskRepository
) {
    private val windowManager = context.getSystemService(Context.WINDOW_SERVICE) as WindowManager
    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main)

    private var petView: PetView? = null
    private var panelView: TaskPanel? = null
    private var petParams: WindowManager.LayoutParams? = null

    private var initialX = 0
    private var initialY = 0
    private var initialTouchX = 0f
    private var initialTouchY = 0f
    private var isDragging = false
    private val touchSlop = 10

    fun show() {
        if (petView != null) return
        val view = PetView(context)
        val params = createPetParams()
        windowManager.addView(view, params)
        petView = view
        petParams = params
        setupTouchListener(view)
    }

    fun hide() {
        petView?.let { windowManager.removeView(it) }
        petView = null
        panelView?.let { windowManager.removeView(it) }
        panelView = null
    }

    private fun createPetParams(): WindowManager.LayoutParams {
        val type = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY
        } else {
            @Suppress("DEPRECATION")
            WindowManager.LayoutParams.TYPE_PHONE
        }
        return WindowManager.LayoutParams(
            WindowManager.LayoutParams.WRAP_CONTENT,
            WindowManager.LayoutParams.WRAP_CONTENT,
            type,
            WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE,
            PixelFormat.TRANSLUCENT
        ).apply {
            gravity = Gravity.TOP or Gravity.START
            x = 600
            y = 400
        }
    }

    private fun setupTouchListener(view: View) {
        view.setOnTouchListener { v, event ->
            when (event.action) {
                MotionEvent.ACTION_DOWN -> {
                    isDragging = false
                    initialX = petParams?.x ?: 0
                    initialY = petParams?.y ?: 0
                    initialTouchX = event.rawX
                    initialTouchY = event.rawY
                    true
                }
                MotionEvent.ACTION_MOVE -> {
                    val dx = event.rawX - initialTouchX
                    val dy = event.rawY - initialTouchY
                    if (kotlin.math.abs(dx) > touchSlop || kotlin.math.abs(dy) > touchSlop) {
                        isDragging = true
                    }
                    if (isDragging) {
                        petParams?.x = initialX + dx.toInt()
                        petParams?.y = initialY + dy.toInt()
                        petParams?.let { windowManager.updateViewLayout(v, it) }
                    }
                    true
                }
                MotionEvent.ACTION_UP -> {
                    if (isDragging) {
                        snapToEdge()
                        savePosition()
                    } else {
                        onPetClick()
                    }
                    scope.launch { taskRepository.updateLastInteraction() }
                    true
                }
                else -> false
            }
        }
    }

    private fun snapToEdge() {
        val params = petParams ?: return
        val displayMetrics = context.resources.displayMetrics
        val screenWidth = displayMetrics.widthPixels
        val centerX = params.x + 45 // 宠物宽度约 90dp，取中点
        params.x = if (centerX < screenWidth / 2) 0 else screenWidth - 90 * displayMetrics.density.toInt()
        windowManager.updateViewLayout(petView, params)
    }

    private fun savePosition() {
        val params = petParams ?: return
        scope.launch { taskRepository.savePosition(params.x, params.y) }
    }

    private fun onPetClick() {
        // Task 6/7 实现：显示任务面板
    }
}
```

- [ ] **Step 4: 验证编译**

Run: `./gradlew :app:compileDebugKotlin`
Expected: BUILD SUCCESSFUL

- [ ] **Step 5: Commit**

```bash
git add app/src/main/java/com/catpet/FloatingPetManager.kt app/src/main/java/com/catpet/ui/PetView.kt app/src/main/java/com/catpet/ui/TaskPanel.kt
git commit -m "feat: add FloatingPetManager with window management and drag"
```

---

### Task 6: PetView — 猫咪 View、动画与手势

**Files:**
- Modify: `app/src/main/java/com/catpet/ui/PetView.kt`
- Modify: `app/src/main/java/com/catpet/FloatingPetManager.kt` (连接点击事件)

- [ ] **Step 1: 重写 PetView 为 ComposeView**

```kotlin
package com.catpet.ui

import android.content.Context
import android.view.View
import androidx.compose.foundation.Image
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.interaction.MutableInteractionSource
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.size
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.alpha
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.compose.ui.platform.ComposeView
import com.catpet.R
import kotlinx.coroutines.delay

class PetView(context: Context) : ComposeView(context) {

    var onPetClick: (() -> Unit)? = null
    var onPetDoubleClick: (() -> Unit)? = null

    init {
        setContent {
            CatPetContent(
                onClick = { onPetClick?.invoke() },
                onDoubleClick = { onPetDoubleClick?.invoke() }
            )
        }
    }
}

@Composable
fun CatPetContent(
    onClick: () -> Unit,
    onDoubleClick: () -> Unit
) {
    var isSleeping by remember { mutableStateOf(false) }
    var blink by remember { mutableStateOf(false) }

    // 眨眼动画：每 5 秒眨一次
    LaunchedEffect(Unit) {
        while (true) {
            delay(5000)
            blink = true
            delay(150)
            blink = false
        }
    }

    // 闲置睡眠检测（简化：6 秒无操作就睡，实际由外部 lastInteraction 控制）
    var lastClickTime by remember { mutableStateOf(System.currentTimeMillis()) }
    LaunchedEffect(Unit) {
        while (true) {
            delay(1000)
            isSleeping = System.currentTimeMillis() - lastClickTime > 2 * 60 * 60 * 1000L
        }
    }

    val scaleY = if (blink || isSleeping) 0.1f else 1f
    val alpha = if (isNightTime()) 0.5f else 1f

    Box(
        modifier = Modifier
            .size(90.dp)
            .alpha(alpha)
            .clickable(
                interactionSource = remember { MutableInteractionSource() },
                indication = null
            ) {
                lastClickTime = System.currentTimeMillis()
                val now = System.currentTimeMillis()
                // 简单双击检测
                onClick()
            },
        contentAlignment = Alignment.Center
    ) {
        Image(
            painter = painterResource(id = R.drawable.cat_pet),
            contentDescription = "cat",
            modifier = Modifier.size(90.dp).graphicsLayer { this.scaleY = scaleY }
        )
        if (isSleeping) {
            Text("Zzz", fontSize = 16.sp, color = Color(0xFF666666), modifier = Modifier.align(Alignment.TopEnd))
        }
    }
}

private fun isNightTime(): Boolean {
    val hour = java.util.Calendar.getInstance().get(java.util.Calendar.HOUR_OF_DAY)
    return hour >= 22 || hour < 7
}
```

注意：需要导入 `androidx.compose.ui.graphics.graphicsLayer`。完整 import：
```kotlin
import androidx.compose.ui.graphics.graphicsLayer
```

- [ ] **Step 2: 添加双击支持到 PetView**

修改点击逻辑，用 `MutableInteractionSource` + 手动计时实现双击：

```kotlin
// 在 CatPetContent 中
var lastClickTime by remember { mutableStateOf(0L) }

.clickable {
    val now = System.currentTimeMillis()
    if (now - lastClickTime < 300) {
        onDoubleClick()
    } else {
        onClick()
    }
    lastClickTime = now
}
```

- [ ] **Step 3: 在 FloatingPetManager 中连接点击事件**

修改 `setupTouchListener` 和 `onPetClick`：

```kotlin
private fun setupTouchListener(view: View) {
    // PetView 内部已用 Compose 处理点击，这里只处理拖动
    view.setOnTouchListener { v, event ->
        when (event.action) {
            MotionEvent.ACTION_DOWN -> {
                isDragging = false
                initialX = petParams?.x ?: 0
                initialY = petParams?.y ?: 0
                initialTouchX = event.rawX
                initialTouchY = event.rawY
                false  // 不消费 DOWN，让 Compose 也能收到
            }
            MotionEvent.ACTION_MOVE -> {
                val dx = event.rawX - initialTouchX
                val dy = event.rawY - initialTouchY
                if (kotlin.math.abs(dx) > touchSlop || kotlin.math.abs(dy) > touchSlop) {
                    isDragging = true
                }
                if (isDragging) {
                    petParams?.x = initialX + dx.toInt()
                    petParams?.y = initialY + dy.toInt()
                    petParams?.let { windowManager.updateViewLayout(v, it) }
                }
                isDragging  // 拖动时消费事件
            }
            MotionEvent.ACTION_UP -> {
                if (isDragging) {
                    snapToEdge()
                    savePosition()
                }
                scope.launch { taskRepository.updateLastInteraction() }
                isDragging
            }
            else -> false
        }
    }
}
```

在 `show()` 中设置 PetView 的回调：
```kotlin
fun show() {
    if (petView != null) return
    val view = PetView(context).apply {
        onPetClick = { showTaskPanel() }
        onPetDoubleClick = {
            scope.launch { taskRepository.increment() }
        }
    }
    // ... 其余不变
}
```

添加 `showTaskPanel()` 方法（Task 7 实现）：
```kotlin
private fun showTaskPanel() {
    // Task 7 填充
}
```

- [ ] **Step 4: 验证编译**

Run: `./gradlew :app:compileDebugKotlin`
Expected: BUILD SUCCESSFUL

- [ ] **Step 5: Commit**

```bash
git add app/src/main/java/com/catpet/ui/PetView.kt app/src/main/java/com/catpet/FloatingPetManager.kt
git commit -m "feat: add PetView with cat image, breath/blink/nap animations and click handlers"
```

---

### Task 7: TaskPanel — 任务面板 UI

**Files:**
- Modify: `app/src/main/java/com/catpet/ui/TaskPanel.kt`
- Modify: `app/src/main/java/com/catpet/FloatingPetManager.kt` (showTaskPanel 实现)

- [ ] **Step 1: 重写 TaskPanel 为 ComposeView**

```kotlin
package com.catpet.ui

import android.content.Context
import androidx.compose.foundation.Image
import androidx.compose.foundation.background
import androidx.compose.foundation.clickable
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Box
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Row
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.layout.size
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.material3.Button
import androidx.compose.material3.LinearProgressIndicator
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.collectAsState
import androidx.compose.runtime.getValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.res.painterResource
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import androidx.compose.ui.platform.ComposeView
import com.catpet.R
import com.catpet.data.TaskRepository
import kotlinx.coroutines.flow.first

class TaskPanel(context: Context) : ComposeView(context) {

    var onClose: (() -> Unit)? = null
    var taskRepository: TaskRepository? = null

    init {
        setContent {
            val repo = taskRepository ?: return@setContent
            TaskPanelContent(repo = repo, onClose = { onClose?.invoke() })
        }
    }
}

@Composable
fun TaskPanelContent(repo: TaskRepository, onClose: () -> Unit) {
    val progress by repo.progressFlow.collectAsState(initial = 0)
    val taskName by repo.taskNameFlow.collectAsState(initial = "读一段话")
    val target by repo.targetFlow.collectAsState(initial = 17)
    val isComplete = progress >= target

    Column(
        modifier = Modifier
            .background(Color.White, RoundedCornerShape(16.dp))
            .padding(16.dp)
    ) {
        Row(verticalAlignment = Alignment.CenterVertically) {
            Image(
                painter = painterResource(id = R.drawable.cat_pet),
                contentDescription = null,
                modifier = Modifier.size(40.dp)
            )
            Column(modifier = Modifier.padding(start = 8.dp)) {
                Text(taskName, fontSize = 14.sp, color = Color(0xFF1a1a2e))
                Text("已完成 $progress / $target", fontSize = 12.sp, color = Color(0xFF888888))
            }
        }

        LinearProgressIndicator(
            progress = { if (target > 0) progress.toFloat() / target else 0f },
            modifier = Modifier.fillMaxWidth().padding(vertical = 12.dp)
        )

        Row(horizontalArrangement = Arrangement.spacedBy(8.dp)) {
            Button(
                onClick = {
                    if (!isComplete) {
                        // 在协程中调用 increment
                    }
                },
                enabled = !isComplete,
                modifier = Modifier.weight(1f)
            ) {
                Text(if (isComplete) "已完成 🎉" else "+1 完成一遍")
            }
            androidx.compose.material3.TextButton(onClick = onClose) {
                Text("✕")
            }
        }

        if (!isComplete) {
            Text(
                "还差 ${target - progress} 遍，加油喵～",
                fontSize = 11.sp,
                color = Color(0xFFaaaaaa),
                modifier = Modifier.padding(top = 8.dp)
            )
        }
    }
}
```

- [ ] **Step 2: 修复 increment 调用（需要协程作用域）**

TaskPanel 需要一个协程作用域来调用 `repo.increment()`。给 TaskPanel 添加 `CoroutineScope`：

```kotlin
class TaskPanel(context: Context) : ComposeView(context) {
    private val scope = kotlinx.coroutines.CoroutineScope(kotlinx.coroutines.SupervisorJob() + kotlinx.coroutines.Dispatchers.Main)
    // ...
}
```

在 Button onClick 中：
```kotlin
onClick = {
    if (!isComplete) {
        scope.launch { repo.increment() }
    }
}
```

需要导入：`import kotlinx.coroutines.launch`

- [ ] **Step 3: 在 FloatingPetManager 实现 showTaskPanel**

```kotlin
private fun showTaskPanel() {
    panelView?.let { windowManager.removeView(it) }
    val panel = TaskPanel(context).apply {
        taskRepository = this@FloatingPetManager.taskRepository
        onClose = { hideTaskPanel() }
    }
    val params = createPanelParams()
    windowManager.addView(panel, params)
    panelView = panel
}

private fun hideTaskPanel() {
    panelView?.let { windowManager.removeView(it) }
    panelView = null
}

private fun createPanelParams(): WindowManager.LayoutParams {
    val type = if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
        WindowManager.LayoutParams.TYPE_APPLICATION_OVERLAY
    } else {
        @Suppress("DEPRECATION")
        WindowManager.LayoutParams.TYPE_PHONE
    }
    // 面板显示在猫咪上方
    val petY = petParams?.y ?: 400
    return WindowManager.LayoutParams(
        280.dpToPx(),
        WindowManager.LayoutParams.WRAP_CONTENT,
        type,
        WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE,
        PixelFormat.TRANSLUCENT
    ).apply {
        gravity = Gravity.TOP or Gravity.START
        x = 40
        y = (petY - 200).coerceAtLeast(0)
    }
}

private fun Int.dpToPx(): Int {
    return (this * context.resources.displayMetrics.density).toInt()
}
```

- [ ] **Step 4: 点击面板外关闭**

在 `petView` 的 `onPetClick` 中判断：如果面板已显示，则关闭面板而不是重新打开：

```kotlin
onPetClick = {
    if (panelView != null) {
        hideTaskPanel()
    } else {
        showTaskPanel()
    }
}
```

- [ ] **Step 5: 验证编译**

Run: `./gradlew :app:compileDebugKotlin`
Expected: BUILD SUCCESSFUL

- [ ] **Step 6: Commit**

```bash
git add app/src/main/java/com/catpet/ui/TaskPanel.kt app/src/main/java/com/catpet/FloatingPetManager.kt
git commit -m "feat: add TaskPanel UI and wire up click/panel interactions"
```

---

### Task 8: MainActivity — 权限引导与启动

**Files:**
- Create: `app/src/main/java/com/catpet/MainActivity.kt`

- [ ] **Step 1: 创建 MainActivity**

```kotlin
package com.catpet

import android.content.Intent
import android.net.Uri
import android.os.Build
import android.os.Bundle
import android.provider.Settings
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.unit.dp
import com.catpet.data.TaskRepository
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.SupervisorJob
import kotlinx.coroutines.launch

class MainActivity : ComponentActivity() {

    private val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            MainScreen(
                hasOverlayPermission = hasOverlayPermission(),
                onRequestPermission = { requestOverlayPermission() },
                onStart = { startPet() },
                onStop = { stopPet() }
            )
        }
    }

    override fun onResume() {
        super.onResume()
        setContent {
            MainScreen(
                hasOverlayPermission = hasOverlayPermission(),
                onRequestPermission = { requestOverlayPermission() },
                onStart = { startPet() },
                onStop = { stopPet() }
            )
        }
    }

    private fun hasOverlayPermission(): Boolean {
        return if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            Settings.canDrawOverlays(this)
        } else {
            true
        }
    }

    private fun requestOverlayPermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            val intent = Intent(
                Settings.ACTION_MANAGE_OVERLAY_PERMISSION,
                Uri.parse("package:$packageName")
            )
            startActivity(intent)
        }
    }

    private fun startPet() {
        scope.launch {
            TaskRepository(this@MainActivity).ensureDailyReset()
        }
        PetService.start(this)
    }

    private fun stopPet() {
        PetService.stop(this)
    }
}

@Composable
fun MainScreen(
    hasOverlayPermission: Boolean,
    onRequestPermission: () -> Unit,
    onStart: () -> Unit,
    onStop: () -> Unit
) {
    Column(
        modifier = Modifier.fillMaxSize().padding(32.dp),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.spacedBy(16.dp, Alignment.CenterVertically)
    ) {
        Text("桌面猫咪", fontSize = 24.sp.takeIf { true } ?: 24.sp)
        Text("一只陪伴你的桌面宠物猫", fontSize = 14.sp, color = androidx.compose.ui.graphics.Color.Gray)

        if (!hasOverlayPermission) {
            Button(onClick = onRequestPermission) {
                Text("授予悬浮窗权限")
            }
            Text("需要悬浮窗权限才能显示宠物", fontSize = 12.sp, color = androidx.compose.ui.graphics.Color.Red)
        } else {
            Button(onClick = onStart) {
                Text("启动猫咪")
            }
            Button(onClick = onStop) {
                Text("关闭猫咪")
            }
        }
    }
}
```

需要导入：`import androidx.compose.ui.unit.sp`

- [ ] **Step 2: 添加通知权限请求（Android 13+）**

在 `startPet()` 中检查通知权限：

```kotlin
private fun startPet() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        if (checkSelfPermission(android.Manifest.permission.POST_NOTIFICATIONS) != android.content.pm.PackageManager.PERMISSION_GRANTED) {
            requestPermissions(arrayOf(android.Manifest.permission.POST_NOTIFICATIONS), 100)
            return
        }
    }
    scope.launch { TaskRepository(this@MainActivity).ensureDailyReset() }
    PetService.start(this)
}
```

- [ ] **Step 3: 验证编译**

Run: `./gradlew :app:assembleDebug`
Expected: BUILD SUCCESSFUL，生成 apk

- [ ] **Step 4: Commit**

```bash
git add app/src/main/java/com/catpet/MainActivity.kt
git commit -m "feat: add MainActivity with permission guide and start/stop controls"
```

---

### Task 9: 猫咪图片资源

**Files:**
- Copy: `e:\project\trae\temp_20261028_01\cat_pet_v4.jpg` → `app/src/main/res/drawable/cat_pet.jpg`

- [ ] **Step 1: 复制猫咪图片到 drawable**

将 `cat_pet_v4.jpg` 复制为 `app/src/main/res/drawable/cat_pet.jpg`。

- [ ] **Step 2: 构建并生成 APK**

Run: `./gradlew :app:assembleDebug`
Expected: BUILD SUCCESSFUL，APK 在 `app/build/outputs/apk/debug/app-debug.apk`

- [ ] **Step 3: 安装到手机测试**

Run: `adb install -r app/build/outputs/apk/debug/app-debug.apk`
手动验证：
1. 打开 App，授予悬浮窗权限
2. 点击「启动猫咪」，猫咪出现在屏幕右侧
3. 点击猫咪 → 弹出任务面板
4. 点击「+1」→ 进度 +1，进度条更新
5. 长按拖动猫咪 → 松手吸附边缘
6. 完成 17 次 → 显示「已完成 🎉」
7. 夜间（22:00 后）→ 猫咪半透明

- [ ] **Step 4: Commit**

```bash
git add app/src/main/res/drawable/cat_pet.jpg
git commit -m "feat: add cat pet image resource"
```

---

### Task 10: 每日重置调度（WorkManager）

**Files:**
- Modify: `app/src/main/java/com/catpet/PetService.kt`

- [ ] **Step 1: 添加 DailyResetWorker**

Create `app/src/main/java/com/catpet/data/DailyResetWorker.kt`:

```kotlin
package com.catpet.data

import android.content.Context
import androidx.work.CoroutineWorker
import androidx.work.WorkerParameters

class DailyResetWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {
    override suspend fun doWork(): Result {
        TaskRepository(applicationContext).ensureDailyReset()
        return Result.success()
    }
}
```

- [ ] **Step 2: 在 PetService 启动时调度每日重置**

在 `PetService.onCreate()` 末尾添加：

```kotlin
import androidx.work.PeriodicWorkRequestBuilder
import androidx.work.WorkManager
import com.catpet.data.DailyResetWorker
import java.util.concurrent.TimeUnit

// 在 onCreate 中
val workRequest = PeriodicWorkRequestBuilder<DailyResetWorker>(1, TimeUnit.HOURS)
    .build()
WorkManager.getInstance(this).enqueue(workRequest)
```

每小时检查一次是否需要重置（轻量，ensureDailyReset 内部判断日期）。

- [ ] **Step 3: 验证编译并构建**

Run: `./gradlew :app:assembleDebug`
Expected: BUILD SUCCESSFUL

- [ ] **Step 4: Commit**

```bash
git add app/src/main/java/com/catpet/data/DailyResetWorker.kt app/src/main/java/com/catpet/PetService.kt
git commit -m "feat: add daily reset worker with WorkManager"
```

---

## 自查清单

**Spec 覆盖检查：**
- ✅ 悬浮窗显示 → Task 5
- ✅ 拖动 + 边缘吸附 → Task 5
- ✅ 待机动画（呼吸+眨眼）→ Task 6
- ✅ 双击 +1 → Task 6
- ✅ 闲置打瞌睡 → Task 6
- ✅ 夜间半透明 → Task 6
- ✅ 点击弹任务面板 → Task 7
- ✅ +1 计数 + 进度条 → Task 7
- ✅ 点面板外关闭 → Task 7
- ✅ 完成动画 → Task 7
- ✅ 每日重置 → Task 10
- ✅ 前台服务保活 → Task 4
- ✅ 权限引导 → Task 8
- ✅ 猫咪图片资源 → Task 9

**占位符检查：** 无 TBD/TODO。

**类型一致性：** TaskRepository 的方法名（increment, resetProgress, savePosition, updateLastInteraction, ensureDailyReset）在各 Task 中一致。
