# Patients Android MVVM

A learning project: a native Android app (Kotlin + Jetpack Compose) that manages the patients of a medical clinic. I'm building it to learn how an Android app talks to a REST API using **Retrofit**, and how to organize the code with the **MVVM** architecture.

It is the second part of a mobile development workshop. The first part was the backend: a REST API built with Node.js, Express and MongoDB.

## What I'm learning

- How MVVM splits an app into layers (UI, ViewModel, Repository, data source)
- Unidirectional data flow: the ViewModel exposes the state, the UI displays it and sends events back
- Describing HTTP requests with a Retrofit interface and converting JSON to Kotlin classes with kotlinx.serialization
- Running network calls with coroutines without freezing the screen
- Handling loading, success and error states with a sealed interface
- Why the ViewModel keeps the data when the screen rotates
- Reading network requests in Logcat with OkHttp's logging interceptor

## What the app does

- Shows the list of patients (loading indicator, error message, retry button)
- Adds a patient with a form (the server's validation messages are shown to the user)
- Edits a patient by tapping its card
- Deletes a patient
- Filters the list by status: All / Active / Inactive

## Project structure

```
com.example.patients
├── PatientsApplication.kt
├── MainActivity.kt
├── di/AppContainer.kt          builds Retrofit and the Repository
├── data/
│   ├── model/Patient.kt        Patient, PatientRequest, ApiError
│   ├── remote/PatientApi.kt    Retrofit interface
│   └── repository/PatientRepository.kt
└── ui/patients/
    ├── PatientsUiState.kt      Loading / Success / Error
    ├── PatientViewModel.kt
    └── PatientsScreen.kt       Compose UI
```

## Tech used

Kotlin, Jetpack Compose (Material 3), Retrofit, OkHttp, kotlinx.serialization, Coroutines and StateFlow.

## How to run it

1. Start the backend API from the first workshop (`npm run dev`, port 5000) with a few patients in the database.
2. Open this project in Android Studio and sync Gradle.
3. Run the app on the emulator. It reaches the API at `http://10.0.2.2:5000/api/`.

On a real phone, replace `10.0.2.2` in `di/AppContainer.kt` with your computer's IP address (same Wi-Fi), and add that IP to `res/xml/network_security_config.xml`.

## API endpoints used

| Method | URL | Purpose |
|---|---|---|
| GET | `/api/patients?statut=Actif` | List (optional filter) |
| POST | `/api/patients` | Create |
| PUT | `/api/patients/:id` | Update |
| DELETE | `/api/patients/:id` | Delete |

## Status

Work in progress, made for learning. Feedback is welcome.
