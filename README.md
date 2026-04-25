# Jetbrains Extension for Sweep (vibe coded mini fork)

This is a vibe coded mini fork of [sweepai/jetbrains_plugin](https://github.com/sweepai/jetbrains_plugin). It exists only to flip a single build property so the plugin can be installed in JetBrains 2026.1 IDEs after Sweep AI shut down its hosted service. I don't know Java and I don't intend to maintain it.

The original plugin's chat/agent features rely on Sweep's cloud backend, which is no longer reachable. The local autocomplete feature (`uvx sweep-autocomplete` running the `sweep-next-edit-1.5B` model) still works because it does not depend on the cloud.

## Fork changes

### 2026.1 IDE compatibility (`untilBuild` bump)

Bumped `untilBuild` from `253.*` to `261.*` in `build.gradle.kts` so the plugin can be installed in JetBrains 2026.1 IDEs (Rider, IntelliJ IDEA, PyCharm, etc.) without the platform compatibility check rejecting it. Two locations updated:

- `intellijPlatform.pluginVerification.ides` (verifier matrix)
- `tasks.patchPluginXml` (the build that ends up in `plugin.xml`)

This is a compatibility-only change — the plugin is still compiled against IntelliJ Platform 2025.1, so any platform API that was removed or changed in 2026.x may surface as `NoSuchMethodError` / `ClassNotFoundException` at runtime. Run `./gradlew verifyPlugin` to flag those before relying on it.

## Build & install on Windows (step-by-step)

Tested on Windows with Rider 2026.1

### 1. Get a JDK 21 (you cannot use the JDK 25 that ships with current JetBrains IDEs)

The Gradle version this project pins (8.11) cannot run on JDK 25, so you need a separate, slightly older JDK just to build. JDK 21 works.

1. Open <https://github.com/JetBrains/JetBrainsRuntime/releases>.
2. Find a **jbr_jcef-21.** release (e.g. `21.0.10`). Download the ZIP that matches your OS, for example `jbr_jcef-21.0.10-windows-x64-b1163.108.zip`.
3. Extract it somewhere stable — for example `C:\some-path\jbr_jcef-21.0.10-windows-x64-b1163.108\`. Inside you should see a `bin\` folder with `java.exe`. Remember this path.

You don't need to set `JAVA_HOME` or PATH globally — the next step points Gradle at this folder directly.

### 2. Tell Gradle to use that JDK

In the repo root there's a file called `gradle.properties`. Open it and add **one line** at the bottom (use forward slashes even on Windows):

```
org.gradle.java.home=C:/some-path/jbr_jcef-21.0.10-windows-x64-b1163.108
```

Replace the path with where you extracted the JBR. **Do not commit this line** — it's specific to your machine. If you want to share gradle.properties with the upstream repo intact, put this line in `~/.gradle/gradle.properties` (Windows: `C:\Users\<you>\.gradle\gradle.properties`) instead.

### 3. Build the plugin ZIP

From the repo root, in PowerShell or Git Bash:

```
./gradlew buildPlugin
```

The first build takes a while because Gradle downloads the IntelliJ Platform SDK. Subsequent builds are much faster.

When it finishes you'll have:

```
build/distributions/sweepai-1.29.3.zip
```

That's the plugin.

### 4. Install the ZIP in your IDE

1. In Rider / IDEA / PyCharm 2026.1: open **Settings** (`File → Settings`, or `Ctrl+Alt+S`).
2. Go to **Plugins**.
3. Click the gear icon at the top → **Install Plugin from Disk...**
4. Pick `build/distributions/sweepai-1.29.3.zip`.
5. Restart the IDE when prompted.

### 5. Turn on the local autocomplete server

The chat/agent features are dead (the Sweep cloud is offline), but the local autocomplete still works.

1. Open the plugin's settings panel inside the IDE — look for the Sweep section in `Settings → Tools` (the exact label depends on the build).
2. Find the **Account** tab.
3. Tick **Enable Local Autocomplete Server**.
4. Click the **Start Server** button.

A terminal tab will open and run `uvx sweep-autocomplete ...`. The first run downloads `uv`, a Python 3.12 environment, `llama-cpp-python` (CPU build), and the `sweep-next-edit-1.5B` model from Hugging Face — about 1–2 GB total, so it takes a few minutes.

When the status indicator next to the checkbox flips to **Running** (green), open any source file and start typing — ghost-text suggestions should appear in grey. Press **Tab** to accept, **Esc** to dismiss.

If `uv` is not on your `PATH`, the plugin will offer to install it for you. You can also install it ahead of time with `winget install astral-sh.uv` (Windows) or following the instructions at <https://docs.astral.sh/uv/>.

## Original README

Open in IntelliJ and click Run Plugin on the top right corner.
And then download required packages.

## Customizing Autocomplete Keystrokes

The autocomplete accept and reject keystrokes are fully customizable via IntelliJ's keymap settings:

### Default Keystrokes
- **Accept Completion**: `Tab`
- **Reject Completion**: `Escape`

### How to Customize
1. Open **Settings/Preferences** → **Keymap**
2. Search for **"Accept Edit Completion"** or **"Reject Edit Completion"**
3. Right-click on the action and select **"Add Keyboard Shortcut"**
4. Assign your preferred keystroke (e.g., `Enter`, `Ctrl+Space`, etc.)
5. The plugin will automatically adapt to your custom keystrokes without requiring a restart

**Note**: The keystroke must map to a standard editor action (like TAB, ENTER, ESCAPE, arrow keys, etc.) to be intercepted reliably. Custom key combinations that don't correspond to editor actions may not work.

## IdeaVim Integration

For IdeaVim users, put in your `~/.ideavimrc`:

```vim
sethandler <Tab> a:ide
```

Or if it doesn't work:

```vim
map <Tab> :action dev.sweep.assistant.autocomplete.edit.AcceptEditCompletionAction<CR>
```

You can also customize the IdeaVim mapping to use different keys:

```vim
" Use Enter to accept completions
map <CR> :action dev.sweep.assistant.autocomplete.edit.AcceptEditCompletionAction<CR>

" Use Ctrl+Y to reject completions
map <C-y> :action dev.sweep.assistant.autocomplete.edit.RejectEditCompletionAction<CR>
```
