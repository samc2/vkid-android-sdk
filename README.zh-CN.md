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

:information_source: VK ID SDK 2.0.0版本支持授权
协议[OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-10)，以及通过 Odnoklassniki 和邮件帐户登录的方法。

* * *

## 示范

SDK поставляется с [应用测试示例](sample/app)，您可以在其中查看授权的工作原理。
为了成功构建测试应用程序，首先创建一个文件`sample/app/secrets.properties`并在其中写入您的 client_id 和 client_secret
VK ID申请：

文件`secrets.properties`:

    VKIDClientSecret=Ваш защищённый ключ
    VKIDClientID=Ваш ID приложения

## 之前

什么是 VK ID 以及如何将其集成到应用程序中
读[在“入门”一文中](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/android/install).

连接VK ID SDK时，首先需要获取应用ID（app_id）和安全密钥（client_secret）。为此，请创建一个应用程序
V[VK ID连接办公室](https://id.vk.ru/business/go).

提供 API SDK 文档[在 Github 上](https://vkcom.github.io/vkid-android-sdk/).

## 安装

首先，添加存储库：

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

设置占位符插件：

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

并包括库：

```kotlin
implementation("com.vk.id:vkid:${sdkVersion}")
```

并在 部分中写下宣言的占位符`android`V`build.gradle.kts`:

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

## 一体化

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

授权是使用authorize()方法触发的，该方法有两个调用选项：

```kotlin
viewModelScope.launch {
    VKID.instance.authorize(vkAuthCallback)
}
```

或通过 LifecycleOwner 传递：

```kotlin
VKID.instance.authorize(this@MainActivity, vkAuthCallback) // Первый параметр LifecycleOwner, например активити.
```

</details>

<details>
<summary>Параметры aвторизации</summary>

您可以使用辅助构建器函数传递其他授权参数：

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

代币的有效期是有限的；当您收到来自 API 的错误时，请更新它：

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

还有一个带有 LifecycleOwner 转移的版本：

```kotlin
VKID.instance.refreshToken(
    lifecycleOwner = MainActivity@ this,
    callback = ... // такой же, как в suspend версии
)
```

</details>

## 文档

-   [VKID是什么](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/intro/start-page)
-   [创建应用程序](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/create-application)
-   [设计要求](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/guidelines/design-rules-oauth)
-   [一键按钮](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/onetap-button/onetap-android)
-   [授权帘](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/onetap-drawer/floating-onetap-android)
-   [小部件三合一](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/widget-3-1/three-in-one-android)

## 本地构建

如果项目构建时没有出现奇怪的错误，那么很可能您需要在工作室的项目设置中进行设置**jdk-17**。为了工作
从控制台运行gradle脚本，还需要在JAVA_HOME环境变量中设置jdk所在路径**17号**版本。

## 贡献

VK ID SDK 项目在 GitHub 上开源，您可以参与其开发 - 我们将感谢您的改进和改进
纠正可能的错误。

### 贡献指南

在[手动的](CONTRIBUTING.md)您可以详细了解开发流程，了解如何提出改进和修复建议，以及如何
在 VK ID SDK 中添加并测试您的更改。
我们还建议您熟悉一般[правилами оформления кода](CODE_STYLE.md)在项目中和[技术团队名单](TECHNICAL_COMMANDS.md).
