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

:information_source : La version VK ID SDK 2.0.0 prend en charge l'autorisation par
protocole[OAuth 2.1](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-v2-1-10), ainsi que les méthodes de connexion via les comptes Odnoklassniki et Mail.

* * *

## Démonstration

Le SDK est livré avec[exemple de test d'application](sample/app), où vous pouvez voir comment fonctionne l'autorisation.
Pour que l'application de test soit créée avec succès, créez d'abord un fichier`sample/app/secrets.properties`et écrivez-y le client_id et le client_secret de votre
Applications d'identification VK :

Déposer`secrets.properties`:

    VKIDClientSecret=Ваш защищённый ключ
    VKIDClientID=Ваш ID приложения

## Précédemment

Qu'est-ce que VK ID et comment l'intégrer dans l'application
lire[dans l'article "Mise en route"](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/android/install).

Pour connecter le SDK VK ID, obtenez d’abord l’ID d’application (app_id) et la clé sécurisée (client_secret). Pour ce faire, créez une application
V[Bureau de connexion VK ID](https://id.vk.ru/business/go).

Documentation du SDK API disponible[sur GitHub](https://vkcom.github.io/vkid-android-sdk/).

## Installation

Pour commencer, ajoutez les dépôts :

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

Configurez le plugin d'espace réservé :

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

Et incluez la bibliothèque :

```kotlin
implementation("com.vk.id:vkid:${sdkVersion}")
```

Et écrivez les espaces réservés du manifeste dans la section`android`V`build.gradle.kts`:

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

## Intégration

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

L'autorisation est déclenchée à l'aide de la méthode authorize(), qui propose deux options d'appel :

```kotlin
viewModelScope.launch {
    VKID.instance.authorize(vkAuthCallback)
}
```

ou en passant LifecycleOwner :

```kotlin
VKID.instance.authorize(this@MainActivity, vkAuthCallback) // Первый параметр LifecycleOwner, например активити.
```

</details>

<details>
<summary>Параметры aвторизации</summary>

Vous pouvez transmettre des paramètres d'autorisation supplémentaires à l'aide de la fonction Helper Builder :

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

Le jeton a une durée de vie limitée ; lorsque vous recevez une erreur de l'API, mettez-la à jour :

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

Il existe également une version avec transfert LifecycleOwner :

```kotlin
VKID.instance.refreshToken(
    lifecycleOwner = MainActivity@ this,
    callback = ... // такой же, как в suspend версии
)
```

</details>

## Documentation

-   [Qu'est-ce que l'identifiant VK](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/intro/start-page)
-   [Création d'une application](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/create-application)
-   [Exigences de conception](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/guidelines/design-rules-oauth)
-   [Bouton OneTap](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/onetap-button/onetap-android)
-   [Rideau d'autorisation](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/onetap-drawer/floating-onetap-android)
-   [Widget 3en1](https://id.vk.ru/about/business/go/docs/ru/vkid/latest/vk-id/connection/elements/widget-3-1/three-in-one-android)

## Construction locale

Si le projet ne se construit pas avec des erreurs étranges, vous devrez probablement le définir dans les paramètres du projet dans le studio.**jdk-17**. Afin de travailler
scripts Gradle depuis la console, vous devez également définir le chemin où se trouve le jdk dans la variable d'environnement JAVA_HOME**le 17**versions.

## Contribuer

Le projet VK ID SDK est open source sur GitHub et vous pouvez participer à son développement - nous vous serions reconnaissants d'apporter des améliorations et
correction d'éventuelles erreurs.

### Guide de contribution

DANS[manuel](CONTRIBUTING.md)vous pouvez vous familiariser avec le processus de développement en détail et apprendre comment proposer des améliorations et des correctifs, ainsi que comment
ajoutez et testez vos modifications dans le SDK VK ID.
Nous vous recommandons également de vous familiariser avec les informations générales[règles de conception de code](CODE_STYLE.md)dans le projet et[liste des équipes techniques](TECHNICAL_COMMANDS.md).
