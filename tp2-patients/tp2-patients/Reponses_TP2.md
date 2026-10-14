# TP 2 – Réponses aux questions de compréhension

**1. Pourquoi les données de la liste ne sont-elles pas perdues lors d'une rotation de l'écran ?**
La rotation détruit puis recrée l'Activity et l'interface Compose, mais pas le ViewModel : il est conservé avec ses données. La liste est stockée dans `uiState` (un `StateFlow` du ViewModel), pas dans l'écran. Après la rotation, `viewModel(factory = ...)` renvoie le même ViewModel et l'écran affiche à nouveau l'état existant. Comme `loadPatients()` est lancée dans `init`, donc une seule fois à la création, aucune nouvelle requête n'est envoyée (visible dans Logcat).

**2. Pourquoi `_uiState` est-il privé et modifiable, alors que `uiState` est public et en lecture seule ?**
C'est le flux unidirectionnel (UDF) : seul le ViewModel a le droit de changer l'état. L'UI observe `uiState` (type `StateFlow`, sans setter) et signale ses actions par des événements (`loadPatients()`, `deletePatient()`…). Ainsi l'état ne peut pas être modifié de façon incontrôlée depuis plusieurs endroits, l'écran est toujours le reflet exact de l'état, et les bugs sont plus faciles à localiser.

**3. Différence entre un état (`StateFlow`) et un événement (`SharedFlow`) ? Un exemple de chaque dans ce TP.**
Un `StateFlow` a toujours une valeur courante, qu'il conserve et renvoie à tout nouvel observateur : il décrit *ce qu'il faut afficher*. Un `SharedFlow` (sans replay) transmet des événements ponctuels sans les conserver : il décrit *ce qui vient de se passer* et ne doit être consommé qu'une fois.
- État : `uiState` (Loading / Success(liste) / Error), ou `statutFilter`.
- Événement : `messages`, par exemple « Ali Ben Salah ajouté » dans la Snackbar. Après une rotation, ce message ne doit pas réapparaître, ce qui arriverait avec un `StateFlow`.

**4. Que se passe-t-il si `Patient` contient `val age: Int` (non nullable) et que l'API renvoie un patient sans âge ?**
Le champ `age` est facultatif côté serveur : la clé est absente du JSON (ou `null`). `kotlinx.serialization` ne trouve pas de valeur pour une propriété obligatoire sans valeur par défaut et lève une `SerializationException` (`MissingFieldException`). Dans le Repository, `safeCall` l'attrape et renvoie « Réponse inattendue du serveur. » : toute la liste échoue à cause d'un seul patient sans âge. Solution : `val age: Int? = null`.

**5. Distinguez `HttpException` et `IOException` : laquelle obtient-on si le serveur est arrêté ? si le patient n'existe pas ?**
- `HttpException` : le serveur a répondu, avec un code 4xx ou 5xx (400 données invalides, 404, 500…).
- `IOException` : aucune réponse n'est arrivée (serveur arrêté, mauvaise IP, Wi-Fi coupé, délai dépassé).

Serveur arrêté → `IOException` (message « Serveur injoignable… »). Patient inexistant → `HttpException` avec le code 404, car le serveur répond bien : le message « Patient introuvable » vient du corps JSON de l'erreur (`ApiError`). Pour un DELETE, c'est le code qui fait lever l'exception : avec `Response<Unit>`, Retrofit n'en lève pas, d'où le `throw HttpException(response)` du Repository.

**6. Si l'on remplaçait l'API par une base Room, quels fichiers faudrait-il modifier ? Lesquels resteraient identiques ?**
- À modifier : la source de données. `PatientApi.kt` serait remplacée par un DAO Room (plus une `@Entity` et la base), et `PatientRepository.kt` appellerait le DAO au lieu de l'API (ses `safeCall` / `messageFrom` seraient adaptés). `AppContainer.kt` construirait la base Room au lieu de Retrofit.
- Identiques : `PatientViewModel.kt`, `PatientsUiState.kt` et `PatientsScreen.kt`, car le ViewModel ne connaît que le Repository et ses `Result<...>`, et l'écran ne connaît que l'état. Les couches basses peuvent changer sans toucher à l'UI ; c'est tout l'intérêt de MVVM.
