# Сборка Android-приложения

Это нативное Android-приложение (Kotlin) — оболочка, которая открывает ваш
веб-сервис такси в полноэкранном виде, с иконкой на рабочем столе телефона,
доступом к геолокации и без адресной строки браузера.

**Важно:** собрать APK/AAB-файл в облачной песочнице Claude не получилось —
корпоративная сетевая политика блокирует загрузку Android SDK с серверов
Google (dl.google.com), это не техническая ошибка, а осознанное ограничение
безопасности. Поэтому весь код (включая релизную подпись, см. ниже) готов,
но собрать из него установочный файл нужно одним из двух способов ниже —
оба бесплатны и не требуют Mac.

## Два вида сборки

- **Debug** (`assembleDebug`) — для собственного тестирования на телефоне.
  Подписывается временным отладочным ключом Android, который магазины
  приложений (RuStore, Google Play) не принимают.
- **Release** (`assembleRelease` / `bundleRelease`) — то, что реально
  загружается в RuStore/Google Play. Подписывается вашим постоянным
  ключом — см. раздел ниже.

## Настройка релизной подписи (сделано один раз, обязательно перед RuStore)

Claude уже сгенерировал для вас постоянный ключ подписи и настроил
`app/build.gradle.kts` так, чтобы читать его из переменных окружения (сам
пароль нигде не хранится в коде проекта). Вам передан отдельно файл
**`release.keystore.jks`** и файл **`KEYSTORE_SECRETS.txt`** с паролем,
алиасом ключа и предупреждением о том, что их нельзя терять и нельзя
выкладывать в открытый доступ. Также передан **`release.keystore.b64.txt`**
— тот же keystore, но закодированный в base64 одной строкой, для удобной
вставки в GitHub.

### Если собираете через GitHub Actions (способ 1 ниже)

1. В репозитории на GitHub откройте **Settings → Secrets and variables →
   Actions → New repository secret** и создайте четыре секрета:
   - `RELEASE_KEYSTORE_BASE64` — вставьте содержимое файла
     `release.keystore.b64.txt` целиком (это одна длинная строка).
   - `RELEASE_STORE_PASSWORD` — значение `RELEASE_STORE_PASSWORD` из
     `KEYSTORE_SECRETS.txt`.
   - `RELEASE_KEY_ALIAS` — значение `RELEASE_KEY_ALIAS` из того же файла
     (`umka-release`).
   - `RELEASE_KEY_PASSWORD` — значение `RELEASE_KEY_PASSWORD` (у вас оно
     совпадает с паролем хранилища).
2. Больше ничего делать не нужно — обновлённый workflow
   (`.github/workflows/build-android.yml`) сам соберёт и debug-, и
   подписанный release-APK, и release-AAB при каждом пуше в `main`/`master`
   или вручную через **Actions → Run workflow**.
3. После сборки на вкладке **Actions** в результатах появятся три файла:
   `taxi-app-debug`, `taxi-app-release-apk`, `taxi-app-release-aab`. Для
   RuStore нужен один из последних двух (release APK или AAB — оба формата
   RuStore принимает).
4. Если секреты не заданы — шаг сборки release просто пропустится с
   предупреждением, debug-сборка при этом всё равно соберётся как раньше.

### Если собираете локально в Android Studio (способ 2 ниже)

Перед сборкой release задайте переменные окружения в терминале (или в
настройках запуска Android Studio: **Run → Edit Configurations →
Environment variables**):

```
RELEASE_STORE_FILE=/полный/путь/к/release.keystore.jks
RELEASE_STORE_PASSWORD=<значение из KEYSTORE_SECRETS.txt>
RELEASE_KEY_ALIAS=umka-release
RELEASE_KEY_PASSWORD=<то же значение>
```

Затем: **Build → Generate Signed Bundle / APK** либо через терминал —
`gradlew assembleRelease` (APK) или `gradlew bundleRelease` (AAB, из корня
проекта). Файл `release.keystore.jks` держите ВНЕ папки проекта (или
убедитесь, что `.gitignore`, который уже добавлен, не даст ему попасть
в git).

## Способ 1 — GitHub Actions (рекомендуется, ничего устанавливать не нужно)

1. Зарегистрируйтесь на https://github.com (бесплатно), если ещё нет аккаунта.
2. Создайте новый репозиторий (можно приватный — секреты подписи в любом
   случае не попадут в код, они хранятся отдельно в настройках репозитория).
3. Загрузите в него содержимое папки `taxi-android` (через веб-интерфейс
   GitHub — кнопка «Add file» → «Upload files» — или через `git push`).
   Файлы `release.keystore.jks`, `release.keystore.b64.txt` и
   `KEYSTORE_SECRETS.txt` в репозиторий НЕ загружайте — они не нужны в
   коде, а только для вставки в GitHub Secrets (см. выше), и уже добавлены
   в `.gitignore` на всякий случай.
4. **Важно:** папка `.github` — защищённый путь, поэтому в вашей рабочей
   папке на компьютере файл `.github/workflows/build-android.yml` мог не
   создаться автоматически. Проверьте: если его нет — создайте сами. В
   GitHub нажмите «Add file» → «Create new file», в поле имени введите
   ровно `.github/workflows/build-android.yml` (слэши создадут вложенные
   папки автоматически) и вставьте туда содержимое из раздела «Содержимое
   build-android.yml» в конце этого файла. Сохраните (Commit new file).
   Если файл у вас всё же есть (например, вы распаковали присланный
   `taxi-android.zip`, где он точно есть) — просто загрузите его как есть.
5. Настройте секреты, как описано в разделе «Настройка релизной подписи»
   выше.
6. В репозитории откройте вкладку **Actions** — сборка запустится
   автоматически. Если не запустилась сама — нажмите «Run workflow».
7. Когда сборка закончится (обычно 3–5 минут), откройте её результат и
   скачайте нужный файл: `taxi-app-debug` — для теста на своём телефоне,
   `taxi-app-release-aab` или `taxi-app-release-apk` — для загрузки в
   RuStore.
8. Для собственного теста: скопируйте `app-debug.apk` на телефон (USB,
   Google Диск, Telegram «Избранное» и т.п.) и откройте его — Android
   спросит разрешение на установку из неизвестного источника, подтвердите.

## Способ 2 — Android Studio на вашем компьютере

1. Скачайте и установите Android Studio: https://developer.android.com/studio
   (бесплатно, ~1 ГБ, работает на Windows).
2. Откройте Android Studio → «Open» → выберите папку `taxi-android`.
3. Android Studio сам предложит скачать недостающие компоненты SDK и создать
   Gradle wrapper — согласитесь на всё.
4. Дождитесь окончания синхронизации проекта (внизу окна).
5. Для теста на своём телефоне: нажмите зелёный треугольник «Run» (▶) с
   подключённым по USB телефоном (с включённой отладкой по USB).
6. Для публикации: задайте переменные окружения из раздела «Настройка
   релизной подписи» выше, затем **Build → Generate Signed Bundle / APK**.

## Публикация в RuStore — коротко

1. Зарегистрируйте компанию (ООО «НПН-ГРУПП») в RuStore Console —
   потребуется усиленная квалифицированная электронная подпись (УКЭП) и
   реквизиты (ИНН 6200017203, ОГРН 1256200006435).
2. Соберите **release** APK или AAB одним из способов выше — именно
   подписанный, не debug.
3. Заполните карточку приложения: название, иконка 512×512, описания,
   скриншоты, категория, возрастной рейтинг и — обязательно — постоянную
   ссылку на политику конфиденциальности: `https://umkatax.ru/privacy.html`.
4. Загрузите файл и отправьте на модерацию (обычно от нескольких часов до
   нескольких рабочих дней).

## Дальше: iOS

Как договаривались — сборку под iOS я не могу сделать сама (нет своего Mac).
Когда дойдёт очередь до iOS, самый простой вариант — сервис Codemagic
(https://codemagic.io) или Expo EAS Build: они собирают iOS-приложения в
облаке без своего Mac. Скажите, когда будете готовы этим заняться —
подготовлю код и настрою сборку.

## Что можно улучшить потом

- Сейчас это WebView-обёртка над веб-версией — рабочий и быстрый способ
  получить настоящее приложение уже сейчас. Полностью нативный интерфейс
  (более плавный, с нативными картами) можно сделать следующим шагом на
  React Native или Kotlin+Jetpack Compose, когда веб-версия стабилизируется.
- Значок приложения сгенерирован программно — при желании закажите
  дизайнеру красивую иконку и просто замените PNG-файлы в папках
  `app/src/main/res/mipmap-*/`.
- Если также будете публиковаться в Google Play — используйте этот же
  `release.keystore.jks`, чтобы сертификат подписи совпадал в обоих
  магазинах (это упрощает параллельные обновления).

## Содержимое build-android.yml

Если файла `.github/workflows/build-android.yml` у вас нет (папка `.github`
защищена и могла не скопироваться на компьютер), создайте его сами на
GitHub с этим содержимым — оно уже включает и debug-, и release-сборку:

```yaml
name: Build Android APK

on:
  push:
    branches: [ main, master ]
  workflow_dispatch: {}

jobs:
  build-debug:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Set up Android SDK
        uses: android-actions/setup-android@v3

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Build debug APK
        run: gradle assembleDebug

      - name: Upload debug APK
        uses: actions/upload-artifact@v4
        with:
          name: taxi-app-debug
          path: app/build/outputs/apk/debug/app-debug.apk

  # Собирает ПОДПИСАННЫЙ релизный APK и AAB — то, что реально принимают
  # RuStore и Google Play. Запускается, только если в настройках репозитория
  # (Settings → Secrets and variables → Actions) добавлены секреты:
  #   RELEASE_KEYSTORE_BASE64 — файл release.keystore.jks, закодированный в base64
  #   RELEASE_STORE_PASSWORD  — пароль от keystore
  #   RELEASE_KEY_ALIAS       — алиас ключа (umka-release)
  #   RELEASE_KEY_PASSWORD    — пароль ключа (для этого keystore совпадает со storePassword)
  # Подробности и точные значения — в файле KEYSTORE_SECRETS.txt.
  build-release:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: temurin
          java-version: '17'

      - name: Set up Android SDK
        uses: android-actions/setup-android@v3

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4

      - name: Check release secrets are configured
        id: check
        run: |
          if [ -n "${{ secrets.RELEASE_KEYSTORE_BASE64 }}" ] && [ -n "${{ secrets.RELEASE_STORE_PASSWORD }}" ] && [ -n "${{ secrets.RELEASE_KEY_ALIAS }}" ]; then
            echo "ready=true" >> "$GITHUB_OUTPUT"
          else
            echo "ready=false" >> "$GITHUB_OUTPUT"
            echo "::warning::Секреты для релизной подписи не настроены — пропускаю сборку release. См. MOBILE_BUILD.md."
          fi

      - name: Decode keystore
        if: steps.check.outputs.ready == 'true'
        run: echo "${{ secrets.RELEASE_KEYSTORE_BASE64 }}" | base64 -d > app/release.keystore.jks

      - name: Build signed release APK + AAB
        if: steps.check.outputs.ready == 'true'
        env:
          RELEASE_STORE_FILE: release.keystore.jks
          RELEASE_STORE_PASSWORD: ${{ secrets.RELEASE_STORE_PASSWORD }}
          RELEASE_KEY_ALIAS: ${{ secrets.RELEASE_KEY_ALIAS }}
          RELEASE_KEY_PASSWORD: ${{ secrets.RELEASE_KEY_PASSWORD }}
        run: gradle assembleRelease bundleRelease

      - name: Upload release APK
        if: steps.check.outputs.ready == 'true'
        uses: actions/upload-artifact@v4
        with:
          name: taxi-app-release-apk
          path: app/build/outputs/apk/release/app-release.apk

      - name: Upload release AAB
        if: steps.check.outputs.ready == 'true'
        uses: actions/upload-artifact@v4
        with:
          name: taxi-app-release-aab
          path: app/build/outputs/bundle/release/app-release.aab
```
