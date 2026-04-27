# 🔐 Lab Frida Android

Installation et utilisation de **Frida** pour l’analyse dynamique d’une application Android.

## 🎯 Objectif

Ce lab a pour objectif de mettre en place un environnement Frida complet afin de :

- installer le client Frida sur Windows ;
- déployer `frida-server` sur un émulateur Android ;
- établir la connexion entre le PC et Android ;
- injecter un script JavaScript minimal ;
- observer des appels Java et natifs ;
- analyser dynamiquement certaines API liées au réseau, aux fichiers et au stockage local.

## 🧪 Environnement utilisé

- Système d’exploitation : Windows
- Terminal : PowerShell
- Outil principal : Frida
- Version Frida : 17.9.1
- Appareil cible : Android Emulator
- Application analysée : `com.android.settings`
- Architecture Android : `x86_64`

## 📁 Contenu du dépôt

```text
.
├── README.md
├── rapport_frida.pdf
├── scripts/
│   ├── hello.js
│   ├── hello_native.js
│   ├── hook_network.js
│   ├── hook_file.js
│   ├── hook_prefs.js
│   ├── hook_debug.js
│   ├── hook_runtime.js
│   └── hook_file_java.js
```

## ⚙️ Installation du client Frida

Frida a été installé sur Windows avec la commande suivante :

```powershell
python -m pip install --upgrade frida frida-tools
```

Vérification de l’installation :

```powershell
frida --version
frida-ps --version
python -c "import frida; print(frida.__version__)"
```
<img width="958" height="143" alt="Capture d&#39;écran 2026-04-27 113434" src="https://github.com/user-attachments/assets/a52b1de0-4e47-46f6-9a8a-0543b70cfc50" />

Dans ce lab, Frida était installé, mais la commande `frida` n’était pas reconnue directement par PowerShell.  
La solution utilisée a été de lancer Frida avec le chemin complet :

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\Scripts\frida.exe" --version
```

## 📱 Préparation de l’émulateur Android

Vérification de la connexion ADB :

```powershell
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" devices
```
<img width="959" height="205" alt="image" src="https://github.com/user-attachments/assets/f2c9c1c8-9036-4a74-80d1-36189e694e52" />

Identification de l’architecture CPU :

```powershell
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" shell getprop ro.product.cpu.abi
```

Résultat obtenu :

```text
x86_64
```
<img width="965" height="42" alt="image" src="https://github.com/user-attachments/assets/fdf05e24-da82-45e4-911a-3940e3af638a" />

## 🚀 Déploiement de frida-server

Le fichier utilisé est :

```text
frida-server-17.9.1-android-x86_64
```

Il a été renommé en :

```text
frida-server
```
<img width="958" height="243" alt="image" src="https://github.com/user-attachments/assets/1a15e249-cff0-49cd-a711-d9b289ddc21e" />

Copie vers l’émulateur :

```powershell
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" push .\frida-server /data/local/tmp/
```

Ajout des droits d’exécution :

```powershell
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" shell chmod 755 /data/local/tmp/frida-server
```
<img width="960" height="44" alt="image" src="https://github.com/user-attachments/assets/06cceb48-18e1-4dcb-9da9-39c7b3bf79d9" />

Lancement de `frida-server` :

```powershell
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" shell /data/local/tmp/frida-server -l 0.0.0.0
```
<img width="965" height="48" alt="image" src="https://github.com/user-attachments/assets/13783c6f-e60c-4355-bc0e-3877eda2ba66" />

## 🔗 Test de connexion Frida

Redirection des ports :

```powershell
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" forward tcp:27042 tcp:27042
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" forward tcp:27043 tcp:27043
```
<img width="959" height="100" alt="image" src="https://github.com/user-attachments/assets/c92fe434-ddda-4429-b453-5bebcbcdaa88" />

Vérification avec Frida :

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\Scripts\frida-ps.exe" -Uai
```
<img width="958" height="411" alt="image" src="https://github.com/user-attachments/assets/04d1354a-fdb4-41d7-b5f7-95449ee018a9" />

Le résultat affiche la liste des applications Android installées sur l’émulateur.

## 🧩 Injection Java minimale

Le script `hello.js` permet de vérifier que Frida peut accéder au runtime Java de l’application.

```javascript
Java.perform(function () {
    console.log("[+] Frida Java.perform OK");
});
```

Commande d’injection :

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\Scripts\frida.exe" -U -f com.android.settings -l hello.js
```

Résultat obtenu :

```text
[+] Frida Java.perform OK
```
<img width="944" height="360" alt="image" src="https://github.com/user-attachments/assets/00a69f88-0be8-4836-aad3-bb8d7cea3eae" />

## ⚙️ Hook natif minimal

Le script `hello_native.js` permet de localiser `libc.so` et la fonction native `recv`.

```javascript
console.log("[+] Script natif chargé");

const libc = Process.getModuleByName("libc.so");
const recvPtr = libc.getExportByName("recv");

console.log("[+] libc.so trouvée");
console.log("[+] recv trouvée à : " + recvPtr);

Interceptor.attach(recvPtr, {
    onEnter(args) {
        console.log("[+] recv appelée");
    }
});
```

Commande utilisée :

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\Scripts\frida.exe" -U -f com.android.settings -l hello_native.js
```

Résultat obtenu :

```text
[+] Script natif chargé
[+] libc.so trouvée
[+] recv trouvée à : 0x...
```

## 🖥️ Console interactive Frida

Après l’injection, la console interactive Frida permet d’inspecter le processus cible.

Commandes utilisées :

```javascript
Process.arch
Process.platform
Process.id
Process.mainModule
Process.getModuleByName("libc.so")
Process.getModuleByName("libc.so").getExportByName("recv")
Java.available
```

Résultats observés :

```text
Process.arch      => x64
Process.platform  => linux
Process.mainModule => app_process64
Java.available    => true
```

## 🌐 Hook réseau

Le script `hook_network.js` permet de localiser les fonctions réseau natives `connect`, `send` et `recv`.

```javascript
console.log("[+] Hooks réseau chargés");

const libc = Process.getModuleByName("libc.so");

const connectPtr = libc.getExportByName("connect");
const sendPtr = libc.getExportByName("send");
const recvPtr = libc.getExportByName("recv");

console.log("[+] connect trouvée à : " + connectPtr);
console.log("[+] send trouvée à : " + sendPtr);
console.log("[+] recv trouvée à : " + recvPtr);
```

Commande utilisée :

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\Scripts\frida.exe" -U -f com.android.settings -l hook_network.js
```

## 📂 Hook fichiers natifs

Le script `hook_file.js` permet d’observer les fonctions natives `open` et `read`.

```javascript
console.log("[+] Hook fichiers chargé");

const libc = Process.getModuleByName("libc.so");

const openPtr = libc.getExportByName("open");
const readPtr = libc.getExportByName("read");

console.log("[+] open trouvée à : " + openPtr);
console.log("[+] read trouvée à : " + readPtr);
```

Commande utilisée :

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\Scripts\frida.exe" -U -f com.android.settings -l hook_file.js
```

## 💾 Hook SharedPreferences

Le script `hook_prefs.js` permet d’observer certaines lectures dans les préférences partagées Android.

```javascript
Java.perform(function () {
    console.log("[+] Hook SharedPreferences chargé");

    var Impl = Java.use("android.app.SharedPreferencesImpl");

    var getString = Impl.getString.overload("java.lang.String", "java.lang.String");
    getString.implementation = function (key, defValue) {
        var result = getString.call(this, key, defValue);
        console.log("[SharedPreferences][getString] key=" + key + " => " + result);
        return result;
    };
});
```

Commande utilisée :

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\Scripts\frida.exe" -U -f com.android.settings -l hook_prefs.js
```

## 🛡️ Hook Debug

Le script `hook_debug.js` permet d’observer certaines méthodes liées à la détection de débogage.

```javascript
Java.perform(function () {
    console.log("[+] Hook Debug chargé");

    var Debug = Java.use("android.os.Debug");

    var isDebuggerConnected = Debug.isDebuggerConnected;
    isDebuggerConnected.implementation = function () {
        var result = isDebuggerConnected.call(Debug);
        console.log("[Debug] isDebuggerConnected() => " + result);
        return result;
    };
});
```

Commande utilisée :

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\Scripts\frida.exe" -U -f com.android.settings -l hook_debug.js
```

Résultat obtenu :

```text
[+] Hook Debug chargé
```

## 📁 Hook chemins de fichiers Java

Le script `hook_file_java.js` a permis d’observer les chemins de fichiers manipulés par l’application.

```javascript
Java.perform(function () {
    console.log("[+] Hook File Java chargé");

    var File = Java.use("java.io.File");

    var initString = File.$init.overload("java.lang.String");
    initString.implementation = function (path) {
        console.log("[File][new] chemin : " + path);
        return initString.call(this, path);
    };

    var exists = File.exists;
    exists.implementation = function () {
        var result = exists.call(this);
        console.log("[File][exists] " + this.getAbsolutePath() + " => " + result);
        return result;
    };
});
```

Commande utilisée :

```powershell
& "$env:LOCALAPPDATA\Programs\Python\Python312\Scripts\frida.exe" -U -f com.android.settings -l hook_file_java.js
```

Résultats observés :

```text
[+] Hook File Java chargé
[File][exists] /data/user_de/0/com.android.settings => true
[File][exists] /data/user_de/0/com.android.settings/cache => true
[File][exists] /data/user_de/0/com.android.settings/shared_prefs => true
[File][new] chemin : /system/etc/security/cacerts
[File][new] chemin : /data/user_de/0/com.android.settings/shared_prefs/battery_fix_prefs.xml.bak
```

## 🔍 Résultats obtenus

- Frida a été correctement installé sur Windows.
- `frida-server` a été déployé sur l’émulateur Android.
- La connexion entre le PC et Android a été validée avec `frida-ps -Uai`.
- L’injection Java avec `Java.perform` a fonctionné.
- Le hook natif a permis de localiser `libc.so` et la fonction `recv`.
- La console interactive Frida a permis d’inspecter le processus cible.
- Les hooks Java ont permis d’observer des accès au stockage local.
- Le hook `java.io.File` a montré des accès vers `cache`, `code_cache`, `shared_prefs` et les certificats système.

## 📌 Conclusion

Ce lab a permis de mettre en place un environnement complet d’analyse dynamique Android avec Frida.

Frida a été utilisé pour injecter des scripts JavaScript dans une application Android, observer des fonctions natives, interagir avec le runtime Java et analyser les chemins de fichiers utilisés par l’application.

Ce travail montre que Frida est un outil très utile pour compléter l’analyse statique par une analyse dynamique du comportement réel d’une application Android.

## 👨‍💻 Auteur

Halim  
ENSA Marrakech — Filière GCDSTE
