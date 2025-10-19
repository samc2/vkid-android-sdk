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

:information_source: संस्करण वीके आईडी एसडीके 2.0.0 द्वारा प्राधिकरण का समर्थन करता है
शिष्टाचार[OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-10), साथ ही Odnoklassniki और मेल खातों के माध्यम से लॉगिन विधियाँ।

* * *

## प्रदर्शन

एसडीके के साथ आता है[अनुप्रयोग परीक्षण उदाहरण](sample/app), जहां आप देख सकते हैं कि प्राधिकरण कैसे काम करता है।
परीक्षण एप्लिकेशन को सफलतापूर्वक बनाने के लिए, पहले एक फ़ाइल बनाएं`sample/app/secrets.properties`और उसमें अपना client_id और client_secret लिखें
वीके आईडी आवेदन:

फ़ाइल`secrets.properties`:

    VKIDClientSecret=Ваш защищённый ключ
    VKIDClientID=Ваш ID приложения

## इससे पहले

वीके आईडी क्या है और इसे एप्लिकेशन में कैसे एकीकृत करें
पढ़ना[लेख "आरंभ करना" में](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/android/install).

वीके आईडी एसडीके को कनेक्ट करने के लिए, पहले एप्लिकेशन आईडी (एप\_आईडी) और सुरक्षित कुंजी (क्लाइंट\_सीक्रेट) प्राप्त करें। ऐसा करने के लिए, एक एप्लिकेशन बनाएं
वी[वीके आईडी कनेक्शन कार्यालय](https://id.vk.ru/business/go).

एपीआई एसडीके दस्तावेज़ उपलब्ध है[जीथूब पर](https://vkcom.github.io/vkid-android-sdk/).

## इंस्टालेशन

आरंभ करने के लिए, रिपॉजिटरी जोड़ें:

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

प्लेसहोल्डर प्लगइन सेट करें:

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

और पुस्तकालय शामिल करें:

```kotlin
implementation("com.vk.id:vkid:${sdkVersion}")
```

और अनुभाग में घोषणापत्र के प्लेसहोल्डर लिखें`android`वी`build.gradle.kts`:

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

## एकीकरण

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

प्राधिकरण को अधिकृत() विधि का उपयोग करके ट्रिगर किया जाता है, जिसमें दो कॉल विकल्प होते हैं:

```kotlin
viewModelScope.launch {
    VKID.instance.authorize(vkAuthCallback)
}
```

या जीवनचक्र स्वामी को पार करने के साथ:

```kotlin
VKID.instance.authorize(this@MainActivity, vkAuthCallback) // Первый параметр LifecycleOwner, например активити.
```

</details>

<details>
<summary>Параметры aвторизации</summary>

आप हेल्पर बिल्डर फ़ंक्शन का उपयोग करके अतिरिक्त प्राधिकरण पैरामीटर पास कर सकते हैं:

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

टोकन सीमित समय के लिए रहता है; जब आपको एपीआई से कोई त्रुटि मिले, तो उसे अपडेट करें:

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

लाइफसाइकिलओनर ट्रांसफर वाला एक संस्करण भी है:

```kotlin
VKID.instance.refreshToken(
    lifecycleOwner = MainActivity@ this,
    callback = ... // такой же, как в suspend версии
)
```

</details>

## प्रलेखन

-   [वीके आईडी क्या है?](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/intro/start-page)
-   [एक एप्लिकेशन बनाना](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/create-application)
-   [डिज़ाइन आवश्यकताएँ](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/guidelines/design-rules-oauth)
-   [वनटैप बटन](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/onetap-button/onetap-android)
-   [प्राधिकरण पर्दा](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/onetap-drawer/floating-onetap-android)
-   [विजेट 3in1](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/widget-3-1/three-in-one-android)

## स्थानीय निर्माण

यदि प्रोजेक्ट अजीब त्रुटियों के साथ नहीं बनता है, तो सबसे अधिक संभावना है कि आपको इसे स्टूडियो में प्रोजेक्ट सेटिंग्स में सेट करने की आवश्यकता है**जेडीके-17**. काम करने के क्रम में
कंसोल से ग्रेडेल स्क्रिप्ट, आपको वह पथ भी सेट करना होगा जहां jdk JAVA_HOME पर्यावरण चर में स्थित है**17वाँ**संस्करण.

## योगदान

वीके आईडी एसडीके प्रोजेक्ट गिटहब पर खुला स्रोत है, और आप इसके विकास में शामिल हो सकते हैं - हम सुधार करने के लिए आभारी होंगे और
संभावित त्रुटियों का सुधार.

### योगदान मार्गदर्शिका

में[नियमावली](CONTRIBUTING.md)आप विकास प्रक्रिया से विस्तार से परिचित हो सकते हैं और सीख सकते हैं कि सुधार और समाधान कैसे प्रस्तावित करें, और कैसे
वीके आईडी एसडीके में अपने परिवर्तन जोड़ें और परीक्षण करें।
हम यह भी अनुशंसा करते हैं कि आप स्वयं को सामान्य से परिचित कर लें[कोड डिज़ाइन नियम](CODE_STYLE.md)परियोजना में और[तकनीकी टीमों की सूची](TECHNICAL_COMMANDS.md).
