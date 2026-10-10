# How to turn SUV Music into an APK — the easy way

You don't need Android Studio. You don't need to write any code.
You just need a **free GitHub account** and about 10 minutes.

Everything happens in the cloud. GitHub does the compiling for you and hands
you back a finished APK file you can install on your phone.

---

## Why this is the easiest method

Building an Android app needs a specific set of tools (Java 17, the Android SDK,
Gradle). Installing them on a phone or a PC takes hours and often goes wrong.
GitHub's free build servers **already have all of them installed**, and your
project includes a file that tells GitHub exactly what to do.

You are literally just going to drag and drop a folder into a website.

---

## Step 1 — Make a GitHub account (skip if you have one)

1. Go to **https://github.com/signup**
2. Enter an email, a password, and a username
3. Verify your email

That's it. The free plan includes unlimited builds of *public* repositories
and 2,000 minutes/month for private ones — a build of this app takes about 3.

---

## Step 2 — Create a new repository

1. Go to **https://github.com/new**
2. **Repository name:** `suvmusic`
3. **Description:** `SUV Music Android player` *(optional)*
4. Choose **Public** *(or Private — both work)*
5. **Do NOT** tick "Add a README file" — leave everything else unchecked
6. Click the green **Create repository** button

You now have an empty repository page.

---

## Step 3 — Upload the project files

On the empty repository page you'll see a link that says
**"uploading an existing file"**. Click it.

*(Or go to `https://github.com/YOUR-USERNAME/suvmusic/upload/main`)*

Now:

1. Extract the project archive I gave you — you should end up with a folder
   called **suvmusic** containing `app`, `gradle`, `build.gradle`, etc.
2. **Open that folder**, select **everything inside it** (Ctrl+A / Cmd+A),
   then drag all of it into the GitHub upload box.
   - ⚠️ **Important:** drag the *contents*, not the folder itself.
     GitHub needs to see `app/` and `build.gradle` at the top level.
3. Wait for the file list to finish uploading (46 files — about 15 seconds)
4. Scroll to the bottom, type a message like `first upload`
5. Click **Commit changes**

> 💡 **The `.github` folder is the important one.** It's hidden on some systems.
> On Windows: View → Show → Hidden items. On macOS: press Cmd+Shift+.
> If you can't see it, don't worry — Step 4 below has the manual way.

---

## Step 4 — Confirm the build file is there

Go back to your repository's main page. You should see a folder called
**`.github`** in the file list.

**If it's missing**, create it manually:

1. Click **Add file → Create new file**
2. In the filename box type exactly:
   `.github/workflows/build.yml`
   *(typing the `/` characters automatically creates the folders)*
3. Open `build.yml` from the project I gave you, copy **all** of it
4. Paste it into the big text box
5. Click **Commit changes**

---

## Step 5 — Watch it build

1. Click the **Actions** tab at the top of your repository
2. You'll see a workflow run called something like
   *"first upload"* or *"Build SUV Music APK"*
3. Click it. You'll see a job named **build**
4. Click **build** → watch the log scroll

**It takes about 2–4 minutes.**

- 🟡 Yellow dot = still running
- ✅ Green check = success, APK is ready
- ❌ Red X = something failed → see *If the build fails* below

---

## Step 6 — Download your APK

Once you see the green check:

1. Scroll down on that same build page
2. Find the box called **Artifacts**
3. Click **suvmusic-debug-apk**

It downloads as a `.zip`. Unzip it and inside is **`app-debug.apk`** —
that's your app. 🎉

---

## Step 7 — Install it on your phone

1. Get the APK onto your phone:
   - Download it directly on the phone's browser, **or**
   - Email/Telegram/Google Drive it to yourself
2. Open it with your phone's **Files** app and tap it
3. Android will say *"For your security, your phone is not allowed to install
   unknown apps from this source"*
4. Tap **Settings** in that message → enable
   **"Allow from this source"** → tap **Back**
5. Tap **Install**
6. Open **SUV Music** and grant the music permission

Your music library appears. Tap any song to play it.

> 🔒 The APK is a *debug* build — fine for personal use, but not signed for
> the Play Store. More on that below if you want it.

---

## If the build fails

Don't panic — 90% of failures are one of these:

| Symptom | Fix |
|---|---|
| `gradlew: No such file` or `not found` | Harmless — the workflow falls back to the system Gradle automatically. |
| `SDK location not found` | Make sure `.github/workflows/build.yml` uploaded correctly. |
| A Kotlin compile error | Copy the **red error text** from the log and send it to me — I'll fix the code. |
| `Permission denied` on gradlew | The workflow already runs `chmod +x` — re-run it. |

To re-run: **Actions tab → click the failed run → "Re-run all jobs"** button.

---

## Getting the app onto your phone more easily

### Option A — Scan a QR code (nicest)
The artifact download requires being logged into GitHub. To avoid that:

1. In the build workflow, after the upload step, the APK is also available
   as a workflow artifact — GitHub requires login for these.

**Simplest workaround:** download the artifact on a computer, then send the
APK to your phone via Google Drive / Telegram / WhatsApp "Saved Messages".

### Option B — Install directly from your phone
1. Open GitHub in your phone's browser
2. Log in, go to Actions → your build → Artifacts
3. Download `suvmusic-debug-apk.zip`
4. Unzip with any file manager app
5. Tap the APK and install

---

## Optional — a proper signed release APK

A debug APK is fine for yourself, but if you want to share it or publish it:

1. In Android Studio (or via the `keytool` command) create a **keystore** file
2. Add these as **GitHub Secrets** (Settings → Secrets and variables → Actions):
   - `KEYSTORE_BASE64` — your keystore, base64-encoded
   - `KEYSTORE_PASSWORD`
   - `KEY_ALIAS`
   - `KEY_PASSWORD`
3. Tell me you want a release build and I'll write the signing workflow for you

That's a later step — get the debug APK working first.

---

## Alternative methods (if GitHub feels like too much)

| Method | Difficulty | Needs a computer? |
|---|---|---|
| **GitHub Actions** ⭐ | Easy | No (works from phone) |
| **AppCircle / Codemagic** | Easy | No — they're web builders, but more setup |
| **Android Studio** | Medium | Yes — powerful but a big download |
| **Termux on the phone** | Hard | No — needs ~3 GB of packages and patience |
| **A friend with Android Studio** | Easy for you | Yes — just hand them the folder |

GitHub Actions genuinely is the least painful route.

---

## What to tell me if you get stuck

Send me any one of these and I'll sort it out:

- A **screenshot** of the red error in the Actions log
- The **error text** copied from the log
- The step number where you got confused

I can also walk you through any single step in more detail, or adapt the
workflow if GitHub changes their interface.