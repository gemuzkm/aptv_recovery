# APTV Recovery

**How to downgrade APTV to a previous version on iPhone**

**English** | [Русский](README.ru.md)

---

The App Store offers no official way to roll an app back. However, if the app has ever been downloaded with your Apple ID, its older builds are still stored on Apple's servers. This guide shows how to get the IPA of the version you need with the `ipatool` command-line utility and install it on an iPhone using Apple Configurator.

Tested on: macOS, iPhone 13, iOS 26.7.1, APTV 1.5.13 → 1.5.12.

You download an app that is already in your purchase history, using your own Apple ID. No jailbreak or developer certificates are required.

## Requirements

- A Mac running macOS with [Homebrew](https://brew.sh) installed
- An iPhone and a cable
- An Apple ID that has downloaded APTV at least once (the examples use `user@example.com`; replace it with yours)
- Two-factor authentication enabled (the code is sent to a trusted device)
- 15–20 minutes

## Step 1. Install ipatool

Open Terminal and run:

```bash
brew tap majd/repo
brew install ipatool
```

Verify the installation:

```bash
ipatool --version
```

## Step 2. Sign in to the App Store

```bash
ipatool auth login -e user@example.com
```

Enter your Apple ID password, then the two-factor code sent to your iPhone or Mac. On success you will see `success=true`.

## Step 3. Find the app and its bundle ID

```bash
ipatool search APTV
```

The output lists several similar apps. The right one is published by Suzhou Xiying Guangnian Technology Co., Ltd:

```json
{"bundleID":"com.kimen.aptvpro","id":1630403500,"name":"APTV", ...}
```

Note the bundle ID: `com.kimen.aptvpro`.

## Step 4. List all versions

```bash
ipatool list-versions -b com.kimen.aptvpro
```

The command returns an `externalVersionIdentifiers` array — the internal IDs of all builds, sorted from oldest to newest. The last ID is the current App Store version, the one before it is the previous version, and so on. Example (end of the list):

```text
..., "889242372", "891251494", "892296607", "893159945"
```

## Step 5. Match IDs to versions

The IDs do not correspond to version numbers, so look up the ones you need:

```bash
ipatool get-version-metadata -b com.kimen.aptvpro --external-version-id 892296607
```

Output:

```text
displayVersion=1.5.12 externalVersionID=892296607 releaseDate=2026-09-30T18:22:44Z success=true
```

The command may take 40–60 seconds; this is normal. Check the last two or three IDs until you find the version you need.

| external-version-id | APTV version | Release date |
|---|---|---|
| 893159945 | 1.5.13 (current) | October 2026 |
| 892296607 | 1.5.12 | 30 Sep 2026 |
| 891251494 | previous | — |

## Step 6. Download the version you need

```bash
ipatool download -b com.kimen.aptvpro --external-version-id 892296607 --platform iphone -o ~/Downloads/APTV_1.5.12.ipa
```

The `--external-version-id` flag is required — without it, the latest version is downloaded.

`success=true` means the IPA is in your Downloads folder. The line `purchased=false` is expected: the app was already in your purchase history, so no new "purchase" was needed.

If ipatool reports that there is no license, run the command below (the app is free) and repeat the download:

```bash
ipatool purchase -b com.kimen.aptvpro
```

## Step 7. Install Apple Configurator

Apple Configurator is a free, official Apple utility. Open the Mac App Store, search for "Apple Configurator" (published by Apple) and click "Get". Alternatively, from Terminal if you have `mas` installed:

```bash
mas install 434290957
```

## Step 8. Install the IPA on the iPhone

1. Connect the iPhone with a cable, unlock it, tap "Trust" when prompted and enter your passcode.
2. Launch Apple Configurator and select your iPhone in the "All Devices" window.
3. Click **Add → Apps**.
4. If you are asked to sign in to the App Store, click "Cancel": signing in is not needed for a local file.
5. Click **Choose from my Mac…** and select `~/Downloads/APTV_1.5.12.ipa`.
6. Configurator warns that the app is already installed — choose **Replace**.
7. Wait for the progress bar to finish without disconnecting the cable.

APTV settings and playlists are preserved because the app is replaced under the same bundle ID.

## After installation

1. Open APTV on the iPhone and check that the "About" section shows version 1.5.12.
2. Turn off automatic updates: Settings → App Store → App Updates → off. Otherwise iOS will silently install 1.5.13 again.
3. When checking for updates manually, do not tap "Update All" — skip APTV.
4. Keep the IPA in a safe place (iCloud Drive, an external drive). If the developer removes old builds, you will not be able to download them again, while a saved file can be installed through Configurator at any time.

> **Do not share or publish your IPA.** The file contains your Apple ID e-mail (in `iTunesMetadata.plist`) and installs only on devices signed in with your account.

## Troubleshooting

| Error | Cause and solution |
|---|---|
| iMazing: "Failed to initiate SRP authentication (GSA)" | A known issue in iMazing 3.6.x. Use ipatool + Configurator as described above. |
| `get-version-metadata` returns the same version for different IDs | A known ipatool bug. Rely on `releaseDate` and the order of IDs in the list. |
| Configurator does not see the iPhone | Reconnect the cable, check that the iPhone appears in the Finder sidebar and that you tapped "Trust". |
| Error `0xE8008015` during installation | The IPA was downloaded with a different Apple ID. The iPhone must be signed in to the App Store with the same account used in ipatool. |
| The app does not launch without internet | On first launch iOS verifies the license — connect the iPhone to Wi-Fi. |

## Returning to the latest version

Update APTV from the App Store as usual, or turn automatic updates back on.

## Other apps

This method works for any app in your purchase history — just substitute your own bundle ID from `ipatool search`.

## Disclaimer

- This is an unofficial project. It is not affiliated with, endorsed by or supported by Apple Inc. or the developer of APTV (Suzhou Xiying Guangnian Technology Co., Ltd). All names and trademarks belong to their respective owners.
- The materials are published for informational purposes only and are provided "as is", without warranty of any kind. You follow this guide at your own risk. The author is not liable for data loss, device or app malfunctions, account restrictions or any other consequences.
- You are responsible for complying with Apple's terms of service, the app's license agreement and the laws of your country.
- This guide does not describe hacking, bypassing payment or circumventing technical protection measures, and it does not require a jailbreak. Do not use it to obtain apps you are not entitled to.

## For rights holders

If you are a rights holder or an authorized representative and believe that any material in this repository infringes your rights, please [open an issue](https://github.com/gemuzkm/aptv_recovery/issues/new). Identify the material in question and confirm your authority to act. The disputed materials will be removed as soon as possible after the request is received.

You may also use GitHub's official procedure: [DMCA Takedown Policy](https://docs.github.com/en/site-policy/content-removal-policies/dmca-takedown-policy).
