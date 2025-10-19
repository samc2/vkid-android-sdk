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

:information_source: الإصدار VK ID SDK 2.0.0 يدعم التفويض بواسطة
بروتوكول[أووث 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-10)بالإضافة إلى طرق تسجيل الدخول من خلال حسابات Odnoklassniki والبريد.

* * *

## توضيح

يأتي SDK مع[مثال اختبار التطبيق](sample/app)، حيث يمكنك رؤية كيفية عمل التفويض.
لكي يتم إنشاء تطبيق الاختبار بنجاح، قم أولاً بإنشاء ملف`sample/app/secrets.properties`واكتب فيه Client_id وclient_secret الخاص بك
تطبيقات معرف VK:

ملف`secrets.properties`:

    VKIDClientSecret=Ваш защищённый ключ
    VKIDClientID=Ваш ID приложения

## سابقًا

ما هو معرف VK وكيفية دمجه في التطبيق
يقرأ[في مقال "البدء"](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/android/install).

لتوصيل VK ID SDK، احصل أولاً على معرف التطبيق (app_id) والمفتاح الآمن (client_secret). للقيام بذلك، قم بإنشاء تطبيق
V[مكتب اتصال معرف VK](https://id.vk.ru/business/go).

وثائق API SDK متاحة[على جيثب](https://vkcom.github.io/vkid-android-sdk/).

## تثبيت

للبدء، أضف المستودعات:

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

قم بإعداد البرنامج المساعد للعنصر النائب:

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

وتشمل المكتبة:

```kotlin
implementation("com.vk.id:vkid:${sdkVersion}")
```

واكتب العناصر النائبة للبيان في القسم`android`V`build.gradle.kts`:

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

## اندماج

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

يتم تشغيل التفويض باستخدام طريقة Authorize()، والتي تحتوي على خيارين للاتصال:

```kotlin
viewModelScope.launch {
    VKID.instance.authorize(vkAuthCallback)
}
```

أو مع مرور LifecycleOwner:

```kotlin
VKID.instance.authorize(this@MainActivity, vkAuthCallback) // Первый параметр LifecycleOwner, например активити.
```

</details>

<details>
<summary>Параметры aвторизации</summary>

يمكنك تمرير معلمات ترخيص إضافية باستخدام وظيفة helper builder:

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

الرمز يعيش لفترة محدودة من الوقت؛ عندما تتلقى خطأ من واجهة برمجة التطبيقات (API)، قم بتحديثه:

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

هناك أيضًا إصدار مع نقل LifecycleOwner:

```kotlin
VKID.instance.refreshToken(
    lifecycleOwner = MainActivity@ this,
    callback = ... // такой же, как в suspend версии
)
```

</details>

## التوثيق

-   [ما هو معرف VK](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/intro/start-page)
-   [إنشاء تطبيق](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/create-application)
-   [متطلبات التصميم](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/guidelines/design-rules-oauth)
-   [زر بنقرة واحدة](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/onetap-button/onetap-android)
-   [ستارة التفويض](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/onetap-drawer/floating-onetap-android)
-   [القطعة 3in1](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/widget-3-1/three-in-one-android)

## بناء محلي

إذا لم يتم إنشاء المشروع بأخطاء غريبة، فمن المرجح أن تحتاج إلى ضبطه في إعدادات المشروع في الاستوديو**جدك-17**. من أجل العمل
gradle من وحدة التحكم، تحتاج أيضًا إلى تعيين المسار حيث يوجد jdk في متغير البيئة JAVA_HOME**السابع عشر**الإصدارات.

## المساهمة

يعد مشروع VK ID SDK مفتوح المصدر على GitHub، ويمكنك الانضمام إلى تطويره - وسنكون ممتنين لإجراء التحسينات
تصحيح الأخطاء المحتملة.

### دليل المساهمة

في[يدوي](CONTRIBUTING.md)يمكنك التعرف على عملية التطوير بالتفصيل ومعرفة كيفية اقتراح التحسينات والإصلاحات، وكذلك كيفية اقتراحها
قم بإضافة واختبار تغييراتك في VK ID SDK.
نوصي أيضًا بالتعرف على الجنرال[قواعد تصميم الكود](CODE_STYLE.md)في المشروع و[قائمة الفرق الفنية](TECHNICAL_COMMANDS.md).
