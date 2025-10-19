<div align="center">
  <h1 align="center">
    <img src="logo.svg" width="150" alt="VK ID SDK Logo">
  </h1>
  <p align="center">
    <a href="LICENSE">
      <img src="https://img.shields.io/npm/l/@vkid/sdk?maxAge=3600">
    </a>
    <a href="https://artifactory-external.vkpartner.ru/ui/native/vkid-sdk-android/com/vk/id/">
        <img src="https://img.shields.io/maven-metadata/v?metadataUrl=https%3A%2F%2Fartifactory-external.vkpartner.ru%2Fartifactory%2Fvkid-sdk-android%2Fcom%2Fvk%2Fid%2Fvkid%2Fmaven-metadata.xml"/>
    </a>
  </p>
  <p align="center">
    VK ID SDK — библиотека для авторизации пользователей Android-приложений с помощью аккаунта VK ID.
  </p>
</div>

* * *

:information_source: VK ID SDK 2.0.0版本支持授權協議[OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-10)，以及通過 Odnoklassniki 和郵件帳戶登錄的方法。

* * *

## 示範

SDK自帶[應用測試示例](sample/app)，您可以在其中查看授權的工作原理。
為了成功構建測試應用程序，首先創建一個文件`sample/app/secrets.properties`並在其中寫入您的 client_id 和 client_secret VK ID申請：

文件`secrets.properties`:

    VKIDClientSecret=Ваш защищённый ключ
    VKIDClientID=Ваш ID приложения

## 之前

什麼是 VK ID 以及如何將其集成到應用程序中讀[在“入門”一文中](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/android/install).

連接VK ID SDK時，首先需要獲取應用ID（app_id）和安全密鑰（client_secret）。為此，請創建一個應用程序V[VK ID連接辦公室](https://id.vk.ru/business/go).

提供 API SDK 文檔[在 Github 上](https://vkcom.github.io/vkid-android-sdk/).

## 安裝

首先，添加存儲庫：

```kotlin
pluginManagement {
    repositories {
        ...
        maven(url = "https://artifactory-external.vkpartner.ru/artifactory/vkid-sdk-android/")
        maven(url = "https://artifactory-external.vkpartner.ru/artifactory/maven/")
    }
}
dependencyResolutionManagement {
    repositories {
        ...
        maven {
            url = URI("https://artifactory-external.vkpartner.ru/artifactory/vkid-sdk-android/")
        }
        maven {
            url = URI("https://artifactory-external.vkpartner.ru/artifactory/maven/")
        }
    }
}
```

設置佔位符插件：

```kotlin
// Корневой build.gradle.kts
// Подключение плагина для Manifest Placeholders.
plugins {
    ...
    id("vkid.manifest.placeholders") version "1.1.0" apply true
}
// Добавление значений в Manifest Placeholders.
vkidManifestPlaceholders {
    // Добавьте плейсхолдеры сокращенным способом. Например, vkidRedirectHost будет "vk.ru", а vkidRedirectScheme будет "vk$clientId".
    init(
        clientId = clientId,
        clientSecret = clientSecret,
    )
    // Или укажите значения явно через properties, если не хотите использовать плейсхолдеры.
    vkidRedirectHost = "vk.ru", // Обычно vk.ru.
    vkidRedirectScheme = "vk1233445", // Строго в формате vk{ID приложения}.
    vkidClientId = clientId,
    vkidClientSecret = clientSecret
}
```

```kotlin
// build.gradle.kts app модулей
plugins {
    id("vkid.manifest.placeholders")
}
```

並包括庫：

```kotlin
implementation("com.vk.id:vkid:${sdkVersion}")
```

並在 部分中寫下宣言的佔位符`android`V`build.gradle.kts`:

```kotlin
android {
    //...
    defaultConfig {
        addManifestPlaceholders(
            mapOf(
                "VKIDClientID" to "1233445", // ID вашего приложения (app_id).
                "VKIDClientSecret" to "000000000000", // Ваш защищенный ключ (client_secret).
                "VKIDRedirectHost" to "vk.ru", // Обычно используется vk.ru.
                "VKIDRedirectScheme" to "vk1233445", // Должно быть vk{ID приложения}.
            )
        )
    }
}
```

## 一體化

<details>
<summary>Инициализация VK ID SDK</summary>
Инициализируйте работу VK ID SDK через объект `VKID`.

```kotlin
// В Application
fun onCreate() {
    super.onCreate()
    VKID.init(this)
}
```

</details>
<details>
<summary>Авторизация</summary>
Результат авторизации передается в коллбэк `VKIDAuthCallback`, поэтому его нужно объявить:

```kotlin
private val vkAuthCallback = object : VKIDAuthCallback {
    override fun onAuth(accessToken: AccessToken) {     
        val token = accessToken.token
        //...
    }

    override fun onFail(fail: VKIDAuthFail) {
        when (fail) {
            is VKIDAuthFail.Canceled -> { /*...*/ }
            else -> {
                //...
            }
        }
    }

}

```

授權是使用authorize()方法觸發的，該方法有兩個調用選項：

```kotlin
viewModelScope.launch {
    VKID.instance.authorize(vkAuthCallback)
}
```

或通過 LifecycleOwner 傳遞：

```kotlin
VKID.instance.authorize(this@MainActivity, vkAuthCallback) // Первый параметр LifecycleOwner, например активити.
```

</details>

<details>
<summary>Параметры aвторизации</summary>

您可以使用輔助構建器函數傳遞其他授權參數：

```kotlin
VKID.instance.authorize(
    callback = vkAuthCallback,
    params = VKIDAuthParams {
        scopes = setOf("status", "email")
    }
)
```

</details>

<details>
<summary>Обновление токена</summary>

代幣的有效期是有限的；當您收到來自 API 的錯誤時，請更新它：

```kotlin
viewModelScope.launch {
    VKID.instance.refreshToken(
        callback = object : VKIDRefreshTokenCallback {
            override fun onSuccess(token: AccessToken) {
                // Использование token
            }
            override fun onFail(fail: VKIDRefreshTokenFail) {
                when (fail) {
                    is FailedApiCall -> fail.description // Использование текста ошибки
                    is RefreshTokenExpired -> fail // Это означает, что нужно пройти авторизацию заново
                    is Unauthorized -> fail // Пользователь понимает, что сначала нужно авторизоваться
                }
            }
        }
    )
}
```

還有一個帶有 LifecycleOwner 轉移的版本：

```kotlin
VKID.instance.refreshToken(
    lifecycleOwner = MainActivity@ this,
    callback = ... // такой же, как в suspend версии
)
```

</details>

## 文件

-   [VKID是什麼](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/intro/start-page)
-   [創建應用程序](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/create-application)
-   [設計要求](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/guidelines/design-rules-oauth)
-   [一鍵按鈕](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/onetap-button/onetap-android)
-   [授權簾](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/onetap-drawer/floating-onetap-android)
-   [小部件三合一](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/widget-3-1/three-in-one-android)

## 本地構建

如果項目構建時沒有出現奇怪的錯誤，那麼很可能您需要在工作室的項目設置中進行設置**jdk-17**。為了工作從控制台運行gradle腳本，還需要在JAVA_HOME環境變量中設置jdk所在路徑**17號**版本。

## 貢獻

VK ID SDK 項目在 GitHub 上開源，您可以參與其開發 - 我們將感謝您的改進和改進糾正可能的錯誤。

### 貢獻指南

在[手動的](CONTRIBUTING.md)您可以詳細了解開發流程，了解如何提出改進和修復建議，以及如何在VK ID SDK 中添加並測試您的更改。
我們還建議您熟悉一般[代碼設計規則](CODE_STYLE.md)在項目中和[技術團隊名單](TECHNICAL_COMMANDS.md).
