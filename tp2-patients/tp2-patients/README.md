# TP 2 – Retrofit et MVVM (projet Patients)

Ce dossier contient les fichiers à copier dans le projet Android Studio `Patients`
(package `com.example.patients`, minSdk 24, Kotlin DSL).

## 1. Créer le projet
Android Studio → New Project → Empty Activity → Name `Patients`, package `com.example.patients`, Min SDK API 24, Kotlin DSL.

## 2. Copier les fichiers
Copiez le contenu de `app/src/main/` de ce dossier dans celui de votre projet (écrasez `MainActivity.kt`) :

- `java/com/example/patients/` : `PatientsApplication.kt`, `MainActivity.kt`, `di/`, `data/`, `ui/patients/`
- `res/xml/network_security_config.xml` (remplacez 192.168.1.20 par l'IP de votre PC si vous utilisez un téléphone)

Le dossier `ui/theme/` est celui généré par Android Studio : ne le remplacez pas.

## 3. Dépendances (partie 2.3 du sujet)
`gradle/libs.versions.toml` : ajoutez
```toml
[versions]
retrofit = "3.0.0"
okhttp = "4.12.0"
kotlinxSerializationJson = "1.9.0"

[libraries]
retrofit = { group = "com.squareup.retrofit2", name = "retrofit", version.ref = "retrofit" }
retrofit-kotlinx-serialization = { group = "com.squareup.retrofit2", name = "converter-kotlinx-serialization", version.ref = "retrofit" }
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" }
kotlinx-serialization-json = { group = "org.jetbrains.kotlinx", name = "kotlinx-serialization-json", version.ref = "kotlinxSerializationJson" }
androidx-lifecycle-viewmodel-compose = { group = "androidx.lifecycle", name = "lifecycle-viewmodel-compose", version.ref = "lifecycleRuntimeKtx" }
androidx-lifecycle-runtime-compose = { group = "androidx.lifecycle", name = "lifecycle-runtime-compose", version.ref = "lifecycleRuntimeKtx" }
androidx-compose-material-icons-core = { group = "androidx.compose.material", name = "material-icons-core" }

[plugins]
kotlin-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
```
`build.gradle.kts` (racine), dans `plugins { }` :
```kotlin
alias(libs.plugins.kotlin.serialization) apply false
```
`app/build.gradle.kts` : dans `plugins { }` ajoutez `alias(libs.plugins.kotlin.serialization)`, et dans `dependencies { }` :
```kotlin
implementation(libs.retrofit)
implementation(libs.retrofit.kotlinx.serialization)
implementation(libs.okhttp.logging)
implementation(libs.kotlinx.serialization.json)
implementation(libs.androidx.lifecycle.viewmodel.compose)
implementation(libs.androidx.lifecycle.runtime.compose)
implementation(libs.androidx.compose.material.icons.core)
```
Puis **Sync Now**.

## 4. AndroidManifest.xml
Ajoutez avant `<application>` :
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
et dans la balise `<application` (en gardant le reste) :
```xml
android:name=".PatientsApplication"
android:networkSecurityConfig="@xml/network_security_config"
```

## 5. Lancer
1. API du TP 1 : `npm run dev` (avec quelques patients en base).
2. Lancer l'application dans l'émulateur (adresse `10.0.2.2:5000`).

## Ce qui va au-delà des séances A et B
- **Exercice 1 (PUT)** : toucher une carte ouvre la même boîte de dialogue, pré-remplie ("Modifier le patient").
- **Exercice 2 (filtre)** : trois `FilterChip` Tous / Actif / Inactif ; le filtre est un état du ViewModel et l'URL devient `/api/patients?statut=Actif` (omis pour « Tous »).
- Petits ajouts par rapport au sujet : `PatientRequest` a un champ `statut` facultatif (sinon on ne pourrait pas créer de patient « Inactif » depuis l'application, et le filtre serait peu testable), et le formulaire a un champ Téléphone.

Limite connue : vider un champ facultatif (ex. le médecin) lors d'une modification ne l'efface pas côté serveur, car les valeurs `null` ne sont pas envoyées.
