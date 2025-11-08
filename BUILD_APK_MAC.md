# CEM6 誤結線検出アプリ - APKビルド手順（Mac版）

このドキュメントでは、React NativeアプリからAndroid APKファイルを生成する手順を説明します。

## 📋 前提条件

### 必要なソフトウェア

1. **Node.js & npm** - v16以上推奨
2. **Java Development Kit (JDK) 17** - Gradle 7.6.3用
3. **Android Studio** - Android SDKとエミュレーター
4. **Gradle 7.6.3** - React Native 0.72.0互換

---

## 🛠️ 環境セットアップ

### 1. Java 17のインストール

\`\`\`bash
# Homebrewでインストール
brew install openjdk@17

# 環境変数を設定（~/.zshrcに追加）
export JAVA_HOME=/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home
export PATH="$JAVA_HOME/bin:$PATH"

# 設定を反映
source ~/.zshrc

# バージョン確認
java -version
\`\`\`

### 2. Android SDKの設定

Android Studioをインストール後、SDK Managerで以下をインストール：

- **Android SDK Platform 33** (Android 13.0)
- **Android SDK Build-Tools 33.0.0**
- **Android SDK Command-line Tools**

環境変数を設定（\`~/.zshrc\`に追加）：

\`\`\`bash
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/platform-tools
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
\`\`\`

### 3. Gradleのインストール

\`\`\`bash
# Homebrewでインストール
brew install gradle

# バージョン確認（7.6.3以上を推奨）
gradle --version
\`\`\`

---

## 🔑 署名鍵の生成

APKに署名するためのキーストアを生成します：

\`\`\`bash
cd android/app

keytool -genkeypair \
  -v \
  -storetype PKCS12 \
  -keystore smartmeter-release.keystore \
  -alias smartmeter-key \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000
\`\`\`

**入力情報**（例）：
- パスワード: \`smartmeter2024\`
- 名前: \`Osaki Electric\`
- 組織: \`Osaki Electric Co., Ltd.\`
- 市区町村: \`Tokyo\`
- 都道府県: \`Tokyo\`
- 国コード: \`JP\`

---

## ⚙️ ビルド設定

### 1. \`android/gradle.properties\`の編集

以下の行を追加：

\`\`\`properties
FLIPPER_VERSION=0.125.0
MYAPP_RELEASE_STORE_FILE=smartmeter-release.keystore
MYAPP_RELEASE_KEY_ALIAS=smartmeter-key
MYAPP_RELEASE_STORE_PASSWORD=smartmeter2024
MYAPP_RELEASE_KEY_PASSWORD=smartmeter2024
\`\`\`

### 2. \`android/app/build.gradle\`の編集

\`signingConfigs\`と\`buildTypes\`を設定：

\`\`\`gradle
android {
    ...
    signingConfigs {
        release {
            if (project.hasProperty('MYAPP_RELEASE_STORE_FILE')) {
                storeFile file(MYAPP_RELEASE_STORE_FILE)
                storePassword MYAPP_RELEASE_STORE_PASSWORD
                keyAlias MYAPP_RELEASE_KEY_ALIAS
                keyPassword MYAPP_RELEASE_KEY_PASSWORD
            }
        }
    }
    buildTypes {
        release {
            signingConfig signingConfigs.release
            minifyEnabled enableProguardInReleaseBuilds
            proguardFiles getDefaultProguardFile("proguard-android.txt"), "proguard-rules.pro"
        }
    }
}
\`\`\`

---

## 🏗️ APKビルド実行

### 1. 依存関係のインストール

\`\`\`bash
npm install
\`\`\`

### 2. Gradleラッパーの生成（初回のみ）

\`\`\`bash
cd android
gradle wrapper --gradle-version=7.6.3
cd ..
\`\`\`

### 3. APKのビルド

\`\`\`bash
cd android
./gradlew assembleRelease
\`\`\`

**ビルド成功時の出力例**：
\`\`\`
BUILD SUCCESSFUL in 21s
\`\`\`

### 4. APKファイルの確認

生成されたAPKファイル：
\`\`\`
android/app/build/outputs/apk/release/app-release.apk
\`\`\`

ファイルサイズ確認：
\`\`\`bash
ls -lh android/app/build/outputs/apk/release/app-release.apk
\`\`\`

---

## 📤 GitHub Releasesへのアップロード

### 1. APKファイルのリネーム

\`\`\`bash
cd android/app/build/outputs/apk/release
cp app-release.apk CEM6-MisconnectionDetect-v1.0.0.apk
\`\`\`

### 2. GitHubでリリースを作成

1. https://github.com/IB6644/CEM6-MisconnectionDetect/releases にアクセス
2. **Draft a new release** をクリック
3. タグ: \`v1.0.0\`
4. タイトル: \`CEM6 誤結線検出アプリ v1.0.0\`
5. APKファイルをドラッグ&ドロップ
6. **Publish release** をクリック

---

## 📱 Androidエミュレーターでのテスト

### 1. エミュレーターの起動

Android Studioから：
1. **Device Manager** を開く
2. 既存のデバイスを選択（または新規作成）
3. **▶️ 起動** をクリック

### 2. APKのインストール

\`\`\`bash
# ダウンロードしたAPKファイルの場所に移動
cd ~/Downloads

# エミュレーターにインストール
adb install CEM6-MisconnectionDetect-v1.0.0.apk
\`\`\`

**インストール成功時の出力**：
\`\`\`
Performing Streamed Install
Success
\`\`\`

### 3. アプリの起動

エミュレーター画面でアプリアイコンをタップするか：

\`\`\`bash
# パッケージ名でアプリを起動
adb shell am start -n com.cem6misconnectiondetect/.MainActivity
\`\`\`

### 4. ログの確認

\`\`\`bash
# リアルタイムログ表示
adb logcat | grep -i "ReactNative\|CEM6"
\`\`\`

---

## 🐛 トラブルシューティング

### エラー: \`JAVA_HOME is not set\`

\`\`\`bash
export JAVA_HOME=/opt/homebrew/opt/openjdk@17/libexec/openjdk.jdk/Contents/Home
\`\`\`

### エラー: \`SDK location not found\`

\`android/local.properties\`を作成：
\`\`\`properties
sdk.dir=/Users/YOUR_USERNAME/Library/Android/sdk
\`\`\`

### エラー: \`Gradle version incompatible\`

Gradleバージョンを確認：
\`\`\`bash
cd android
./gradlew --version
\`\`\`

7.6.3以外の場合は再生成：
\`\`\`bash
gradle wrapper --gradle-version=7.6.3
\`\`\`

### エラー: \`No connected devices\`

エミュレーターの起動確認：
\`\`\`bash
adb devices
\`\`\`

出力例：
\`\`\`
List of devices attached
emulator-5554   device
\`\`\`

---

## 📚 参考情報

- **React Native公式**: https://reactnative.dev/docs/signed-apk-android
- **Android Developer**: https://developer.android.com/studio/build/building-cmdline
- **Gradle Documentation**: https://docs.gradle.org/

---

## 📝 バージョン履歴

- **v1.0.0** (2025-01-08) - 初回リリース
  - 検針時接続機能
  - 誤結線検出接続機能
  - 計測接続機能
  - スマートメーター情報表示
