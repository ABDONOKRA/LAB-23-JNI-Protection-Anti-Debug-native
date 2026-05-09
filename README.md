# 📱 LAB 22 — Intégration JNI/NDK dans une Application Android

---

## 🎯 Objectifs pédagogiques

À la fin de ce laboratoire, vous serez capable de :

- Créer un projet Android avec support C++
- Comprendre le rôle du NDK, de CMake et de JNI
- Déclarer et appeler des méthodes natives depuis Java
- Manipuler des types simples et complexes entre Java et C++
- Gérer des erreurs fréquentes comme `UnsatisfiedLinkError`
- Lire les logs natifs dans Logcat
- Appliquer de bonnes pratiques récentes pour JNI, optimisation et sécurité

---

## 🧰 Prérequis

| Élément | Statut requis |
|---------|--------------|
| Android Studio | Installé |
| SDK Android | Configuré |
| NDK | Disponible via SDK Manager |
| CMake | Disponible via SDK Manager |
| LLDB | Disponible via SDK Manager |
| Connaissances de base | Activity, layout XML, Java |

---

## 📐 Concepts fondamentaux

Avant de coder, il est essentiel de distinguer les différentes briques :

| Brique | Rôle |
|--------|------|
| **JNI** | Interface permettant au code Java/Kotlin d'appeler du code natif C/C++ |
| **NDK** | Ensemble d'outils pour utiliser C/C++ dans Android, accès à la compilation native |
| **CMake** | Outil de build pour décrire comment compiler et lier la bibliothèque native |
| **`.so` (lib partagée)** | Code natif empaqueté dans `libnative-lib.so`, chargé via `System.loadLibrary()` |

---

## ⚙️ Étape 1 — Créer le projet

<img width="1165" height="830" alt="image" src="https://github.com/user-attachments/assets/e7d0b6f4-e4a8-4b15-8c4e-5991173d37c6" />

<img width="1112" height="369" alt="image" src="https://github.com/user-attachments/assets/7e69733e-a3ef-43dc-955c-40082f10cb69" />

<img width="1057" height="832" alt="image" src="https://github.com/user-attachments/assets/199bc16e-7fa8-49d5-b5c1-922b828f2881" />

---

## 🗂️ Structure du projet

```
app/src/main/
├── java/com/example/jnidemo/
│   └── MainActivity.java
└── cpp/
    ├── native-lib.cpp
    └── CMakeLists.txt
```

---

## 📄 Étape 2 — Configurer CMakeLists.txt

`app/src/main/cpp/CMakeLists.txt`

<img width="1240" height="556" alt="image" src="https://github.com/user-attachments/assets/117f2307-6b51-4017-bf13-636a747fbe89" />

---

## 💻 Étape 3 — Écrire le code natif C++

### `native-lib.cpp`

```cpp
#include <jni.h>
#include <algorithm>
#include <climits>
#include <android/log.h>

#define LOGI(...) __android_log_print(ANDROID_LOG_INFO, "NativeLib", __VA_ARGS__)
#define LOGE(...) __android_log_print(ANDROID_LOG_ERROR, "NativeLib", __VA_ARGS__)

extern "C" {

JNIEXPORT jstring JNICALL
Java_com_example_jnidemo_MainActivity_helloFromJNI(JNIEnv *env, jobject /* this */) {
    LOGI("helloFromJNI called");
    return env->NewStringUTF("Hello from C++!");
}

JNIEXPORT jint JNICALL
Java_com_example_jnidemo_MainActivity_factorial(JNIEnv *env, jobject /* this */, jint n) {
    LOGI("factorial called with n = %d", n);
    if (n < 0) return -1;
    jint result = 1;
    for (jint i = 2; i <= n; i++) {
        if (result > INT_MAX / i) return -2;
        result *= i;
    }
    return result;
}

JNIEXPORT jstring JNICALL
Java_com_example_jnidemo_MainActivity_reverseString(JNIEnv *env, jobject /* this */, jstring str) {
    const char *input = env->GetStringUTFChars(str, nullptr);
    if (input == nullptr) return nullptr;
    std::string cppStr(input);
    std::reverse(cppStr.begin(), cppStr.end());
    env->ReleaseStringUTFChars(str, input);
    return env->NewStringUTF(cppStr.c_str());
}

JNIEXPORT jint JNICALL
Java_com_example_jnidemo_MainActivity_sumArray(JNIEnv *env, jobject /* this */, jintArray arr) {
    jint *elements = env->GetIntArrayElements(arr, nullptr);
    if (elements == nullptr) return 0;
    jsize length = env->GetArrayLength(arr);
    jint sum = 0;
    for (jsize i = 0; i < length; i++) {
        sum += elements[i];
    }
    env->ReleaseIntArrayElements(arr, elements, JNI_ABORT);
    return sum;
}

}
```

### Explication des includes

| Header | Utilité |
|--------|---------|
| `<jni.h>` | Types JNI (`jstring`, `jint`, `jintArray`…) |
| `<algorithm>` | Fonction `std::reverse` |
| `<climits>` | Constante `INT_MAX` pour détecter l'overflow |
| `<android/log.h>` | Écriture dans Logcat via `LOGI` / `LOGE` |

---

## ☕ Étape 4 — Déclarer les méthodes natives en Java

<img width="1296" height="981" alt="image" src="https://github.com/user-attachments/assets/83c2fe78-709d-408b-9a66-9ee6670e90d1" />

- ✅ Les 4 méthodes `native` sont déclarées
- ✅ Le bloc `static` charge bien `"native-lib"`
- ✅ Les 4 `TextView` sont correctement appelés

---

## 🎨 Étape 5 — Créer le layout XML

<img width="1296" height="981" alt="image" src="https://github.com/user-attachments/assets/6af5567b-e48a-423a-bfd5-a726e3b98e8c" />

- ✅ Les 4 `TextView` ont les IDs : `tvHello`, `tvFact`, `tvReverse`, `tvArray`
- ✅ Les erreurs rouges dans `MainActivity.java` ont disparu

---

## 🚀 Étape 6 — Compiler et exécuter

<img width="594" height="958" alt="image" src="https://github.com/user-attachments/assets/9ef24b53-6a3e-4cfa-8b0b-b45bb9b486c5" />

### Résultats obtenus

| Opération | Résultat | Status |
|-----------|----------|--------|
| Message JNI | `Hello from C++ via JNI !` | ✅ |
| Factorielle | `Factoriel de 10 = 3628800` | ✅ |
| Inversion | `Texte inverse : !lufrewop si JNI` | ✅ |
| Somme tableau | `Somme du tableau = 150` | ✅ |

---

## 📋 Étape 7 — Vérifier les logs dans Logcat

<img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/8de71fe3-cdad-44b5-bd8f-150f07f4e0e5" />

---

## 🧪 Étape 8 — Tests des cas limites

### Test 1 — Valeur négative : `factorial(-5)`

<img width="1409" height="937" alt="image" src="https://github.com/user-attachments/assets/bff4608a-ca84-4a52-93d1-38b069722d64" />

> ✅ Résultat : `code = -1` — le code C++ a bien détecté la valeur négative

---

### Test 2 — Dépassement d'entier : `factorial(20)`

<img width="1409" height="937" alt="image" src="https://github.com/user-attachments/assets/08508d6e-e8a3-4bef-8395-55280e8e4d82" />

> ✅ Résultat : `code = -2` — overflow détecté via `INT_MAX`

---

### Test 3 — Chaîne vide : `reverseString("")`

<img width="1409" height="937" alt="image" src="https://github.com/user-attachments/assets/a9d7187e-84cc-409a-abb9-b235ef56752c" />

> ✅ Résultat : `Texte inverse :` (chaîne vide après les deux points)

---

### Test 4 — Tableau vide : `sumArray({})`

<img width="1409" height="937" alt="image" src="https://github.com/user-attachments/assets/1e9a28cf-6b7e-4333-a55a-20996871ad28" />

> ✅ Résultat : `Somme du tableau = 0`

---

## 📊 Récapitulatif des tests

| Test | Entrée | Résultat attendu | Status |
|------|--------|-----------------|--------|
| Factorielle normale | `factorial(10)` | `3628800` | ✅ |
| Valeur négative | `factorial(-5)` | `code = -1` | ✅ |
| Overflow entier | `factorial(20)` | `code = -2` | ✅ |
| Chaîne vide | `reverseString("")` | chaîne vide | ✅ |
| Tableau vide | `sumArray({})` | `0` | ✅ |

---

## 🧠 Conclusion

Ce LAB démontre comment intégrer du code natif C++ dans une application Android via **JNI** et **NDK**. Les points clés retenus :

- Le **naming JNI** (`Java_package_Class_method`) est strict et doit correspondre exactement au package Java
- La gestion des types (`jstring`, `jintArray`) nécessite un **acquire/release** explicite des ressources
- Les **macros de log** (`LOGI`, `LOGE`) facilitent le débogage côté natif via Logcat
- Les **cas limites** (valeurs négatives, overflow, entrées vides) doivent être gérés côté C++ avant tout retour vers Java
