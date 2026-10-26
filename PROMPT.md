# Google TV cleanup + custom home screen: prompt

Paste everything below the line into Claude Code, unchanged. Run it from the folder that holds this
file **and** `tv-detox-custom.patch`: the patch is the exact home screen. Claude asks for the
device's IP address and your favourite apps along the way.

---

Help me clean up my Google TV device and make it faster, and give it my own home screen.
The device runs Google TV (for example a Chromecast with Google TV, or another Google TV streamer).

You don't know my device's IP address or my favourite apps yet: ask me for each one when the steps
below say so, one question at a time, and wait for my answer. Don't guess either of them.

The file `tv-detox-custom.patch` next to this prompt is the exact home screen I want. Don't redesign
it: apply it as described in step 6.

1. Before you change anything, tell me in plain words:
   this turns off system apps, and that has some risk.
   A few devices got stuck at the logo or kept restarting (Google TV devices like the Chromecast
   more than others), and you're an AI, so you can make mistakes. undo.sh and your undo list turn
   everything back on. Ask me to confirm I want to go ahead.

2. Ask me for my device's IP address. Tell me where to find it: Settings > Network & Internet >
   my Wi-Fi network (or Ethernet), where it's listed as the IP address, like 192.168.1.50. Check that
   what I give you looks like an IP address before you use it; in the rest of this prompt it's
   called <ip>.
   Then connect to my device over Wi-Fi and walk me through anything I need to tap on it:
   - Developer options: Settings > System > About > Android TV OS build, tap it 7 times.
   - Then Settings > System > Developer options: turn on USB debugging, and Network debugging or
     Wireless debugging (whichever the device has).
   - Try `adb connect <ip>:5555` first. If that's refused, use Wireless debugging: have me
     open it, pick "Pair device with pairing code", and run `adb pair <ip>:<pairing port> <code>`,
     then `adb connect <ip>:<port shown on screen>`. Wireless debugging's port changes after every
     restart, so note the port again after step 8.
   - When the device asks "Allow debugging?", I tick "Always allow" and press Allow.
   - Other devices or emulators may be connected to adb: always pass `-s <device>` to every adb
     command, and check the model with `getprop ro.product.model` before changing anything.
   - Once you know the model, suggest a name for this device (like "MiBox S") and make its own folder
     `devices/<name>` in the folder that holds `devices`, or next to this prompt if there is none.
     Everything about this device goes there: undo-list.md, key.jks, the installed tvhome.apk, and a
     README.md with its model, IP address, favourites and the build, install and undo commands. If a
     folder with that name already exists, it belongs to another device: don't use it or change it.
     Ask me for another name to save it under, and wait for my answer.

3. Before changing anything, list every app in plain English in three groups: safe to turn off,
   needed by the device, and apps I use. tv-detox's `candidates.sh` (read-only) helps with the
   verdicts. Check free memory (MemAvailable in /proc/meminfo).

4. Ask me before every change. Turn apps off (`pm disable-user --user 0`) instead of deleting them.
   Before turning each app off, add its package name on its own line to
   `/data/local/tmp/tvhome-disabled.txt` on the device, so undo.sh turns it back on. Also keep a list
   in `undo-list.md` in the device's folder: what you changed and the exact command to undo each change.

5. Never turn off: Google Play services (com.google.android.gms), Google Play, Settings, System UI,
   setup apps (including com.google.android.tungsten.setupwraith), the keyboard, Bluetooth, the
   remote and its services, HDMI-CEC, the TV input and tuner framework, anything Android marks
   PERSISTENT, or anything else the device needs to start. Never turn off Google TV's home screen
   (com.google.android.apps.tv.launcherx) yourself; only install.sh may, and only as described in
   step 6. If you're not sure, leave it on.

6. Install the home screen:
   - Clone https://github.com/Ansub/tv-detox, check out commit `e94a2de`, and apply the patch with
     `git apply tv-detox-custom.patch`. Stop and tell me if it doesn't apply cleanly.
   - Read install.sh and undo.sh before running them.
   - Build `out/tvhome.apk`: on Windows with `sh build-win.sh` (Git Bash; needs Android Studio's Java,
     the Android SDK and Python), on macOS or Linux with `sh build.sh` (needs Java 17, zip and the
     Android SDK). Build with `KEYSTORE="<device folder>/key.jks"`, so the key the build creates is
     kept in the device's folder: updates must be signed with the same key. Copy `out/tvhome.apk`
     there too once it's installed.
   - Ask me for my favourite apps for the bar at the bottom, in the order I want them. To help me
     choose, first show me the apps installed on the device that I can open (by name, A to Z, not
     package names). I can answer with names in any spelling, like "Netflix, Disney Plus, Plex".
   - Match each name to its package on the device (`pm list packages` plus each app's label) and show
     me the list with names and packages, in my order, before you go on. Ask me about any name you
     can't match or that matches more than one app, and tell me about apps that aren't installed.
     I can change favourites on the device later too (hold OK on an app).
   - Run `install.sh <device> "<packages, comma-separated, in my order>"`. It saves the device's own
     home screen first, so undo.sh can put it back.
   - Google TV: by default install.sh leaves Google TV's home screen on, so the Home button may keep
     opening Google TV instead of the new home screen. Making Home open the new one means turning
     Google TV's home screen off (`GOOGLE_TV=1`), which has left some Chromecasts stuck restarting.
     Explain this to me and ask which I want. Only use `GOOGLE_TV=1` if I say yes and I can reconnect
     over USB or port 5555 (Wireless debugging's port changes on every restart, so it can't be relied
     on for recovery). If install.sh then asks about the setup app, ask me separately. install.sh
     reads its yes/no from the keyboard, so after I say yes, pass `YES=1` (and `YES_SETUP=1` only for
     the setup app).
   - What I should see, and what you check with screenshots (`adb exec-out screencap -p`):
     - A full-screen nature photo from Windows Spotlight, with no text, title or credits on it. It
       changes every 10 minutes; Settings > Next background changes it right away.
     - A 24-hour clock at the top right, and nothing else on the photo except the bar.
     - A favourites bar at the bottom, as a carousel: the selected app stays in the middle and the
       row slides. Three apps show in full on each side, the fourth one is faded, and the row doesn't
       wrap around (at the first app, the left side is empty).
     - Every tile looks the same: a rounded glass square with the app's icon. Icons are grey; the
       selected one gets its own colours, grows a little, its tile glows in the app's colour, and its
       name shows below it.
     - Holding OK on a favourite: Move, Remove from favorites, Uninstall (App info for apps that came
       with the device).
     - Down opens All apps: every installed app A to Z after a Settings tile, with names, in a grid of
       eight across. No clock there, no dark bands, and the grid scrolls below the "All apps" title.
       A selected app in the top or bottom row is never cut off. A dot marks favourites. Holding OK:
       Add to favorites (or Remove from favorites), Uninstall (or App info). Back, or Up from the top
       row, goes back.
     - Screen saver "Fade to sleep" (part of the patch; install.sh sets it up): the device never goes
       dark without warning. At its own screen saver time (Settings > Device Preferences > Screen
       saver > When to start), whatever is on screen, like a paused show, slowly fades to completely
       black over 30 seconds, and then the device sleeps. Any key during the fade brings it straight
       back. install.sh saves the device's screen saver and sleep settings to
       `/data/local/tmp/tvhome-dream.txt` first (undo.sh puts them back) and sets sleep to 30 seconds
       after the screen saver time. Check it with `settings get secure screensaver_components`
       (`com.ansub.tvhome/.FadeDream`) and `settings get secure sleep_timeout` (screen_off_timeout +
       30000). It replaces Ambient Mode (com.google.android.backdrop), so that one is safe to turn off.

7. Turn off animations. First save the current values for undo.sh:
   `adb -s <device> shell 'echo "$(settings get global window_animation_scale) $(settings get global transition_animation_scale) $(settings get global animator_duration_scale)" > /data/local/tmp/tvhome-anim.txt'`
   Then set all three to 0.

8. Wait 30 seconds after the last change, then restart the device once and check it starts up
   normally before you say you're done: it finishes booting, is still up a minute later (no restart
   loop), Home opens the home screen I chose, and nothing crashed. Show me free memory before and
   after, and say that a fresh restart always frees some memory.

9. Tell me how to undo everything: `sh undo.sh <ip>` (with my real IP address filled in) from the tv-detox folder (with
   debugging on). It turns every app on the list back on, puts the animation speeds and the screen saver and sleep settings back, removes the
   new home screen and makes the device's own home screen the default again.

10. Once everything is checked, remind me to turn off USB, Network and Wireless debugging (or
    Developer options) on the device. Left on, other devices on my network can ask to control it. I
    only turn it back on for undo.sh or updates.
