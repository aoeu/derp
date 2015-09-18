# List and pick a target API / android version.
android list targets
set TARGET_NUMBER 3

# Make the project.
android create project --gradle --gradle-version 1.2.3 --path .  --target $TARGET_NUMBER --package less.apps --activity Main --name map

# Fix the project beacuse the android tool is inept.
sed -i '' 's/runProguard/minifyEnabled/' build.gradle
sed -i '' 's/1.12-all/2.2.1-all/' gradle/wrapper/gradle-wrapper.properties

# Build and install the project.
./gradlew --info assembleDebug
adb install build/outputs/apk/derp-debug.apk

# https://developer.android.com/tools/publishing/app-signing.html

# Make a key to sign the app with.
keytool -genkey -v -keystore derp-release-key.keystore -alias derp_key -keyalg RSA -keysize 2048 -validity 10000
# Build an unsigned release app.
./gradlew --info assembleRelease
# Sign the app with wih the private key.
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore derp-release-key.keystore build/outputs/apk/derp-release-unsigned.apk derp_key
# Verify the APK is signed.
jarsigner -verify -verbose -certs build/outputs/apk/derp-release-unsigned.apk
# Rename the apk to reflect reality.
mv build/outputs/apk/derp-release-unsigned.apk build/outputs/apk/derp-release-unaligned.apk
# Zipalign the final apk.
zipalign -v 4 build/outputs/apk/derp-release-unaligned.apk build/outputs/apk/derp-release.apk

# Uninstall and reinstall as a smoke test.
adb uninstall less.apps.derp
adb install build/outputs/apk/derp-release.apk

# https://developers.google.com/maps/documentation/android-api/signup

# Dump the SHA-1 (amidst other things) of the signed app.
keytool -list -v -keystore derp-release-key.keystore  -alias derp_key

# Go make a project named something like "derp".  It'll make an alias called a "Project ID".
open 'https://console.developers.google.com/project'
set PROJECT_ID derp-1072
# Go and turn on the maps API.
open "https://console.developers.google.com/project/$PROJECT_ID/apiui/apiview/maps_android_backend/overview"
# Go make an API key based off the app signing. Select Android Key, type in your app package name (less.apps.derp) and the SHA_1 from the signed app.
open 'https://console.developers.google.com/project/derp-1072/apiui/credential'

# Copy the API key and add it to the AndroidManifest.xml
$ sam -d src/main/AndroidManifest.xml 
-. src/main/AndroidManifest.xml
,x/    /c/	/
/<\/activity>/-
a
			<meta-data android:name="com.google.android.geo.API_KEY" android:value="aoeuaoeuaoeuW1R5I5fLS8ptfx5aoeuaoeuaoeu" />
.
w









