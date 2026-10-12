# SOOPERCHARGE for LOOI Robot

![alt text](https://github.com/SlyFox-Asia/looi-soopercharged/blob/ebfa4fc97cf09908a15f0afcbd31393fc01ab911/SOOPERCHARGE.png)

This repository only hosts useful documentation to the autonomous study of the Robot's functioning.
To comply with TangibleFuture's terms of service, PUBLIC sharing of modified files and decompiled source code is paused.
Private sharing through the invite-only SOOPERCHARGE Discord Server is still possible.
Do not talk about this project in official TangibleFuture channels, such as the official LOOI Robot Discord server. You will be warned or banned. Contact me with AI-written messages and I won’t help you.
Being kind and respectful is the key to wonderful work!

From this point on, this README has been co-writed with the help of AI.
All content being read here has then been reviewed multiple times by an human (me).
I ain't got hours to write, please bear with me.

AI tools have been used to speed up and ASSIST (not replace) myself in the reverse engineering of the robot, specifically in parsing masked stuff in the newer versions of the decrypted app and with BLE-related stuff. Everything else is done by hand.

---

## About the app.

The stock LOOI application locks several features behind server-side configurations and hardware sensors. Through targeted Smali code manipulation, I have successfully bypassed some of these limits. As a response to my work, starting from the 2.6.1 update, TangibleFuture has implemented a paid (and expensive) version of the Jiagu 360 packer and obfuscator, making code on those versions much harder to access. Jiagu 360 was already implemented in older versions, but due to its configuration, it was not impacting code access in any way.

Recently this year, after the TangibleFuture team has reviewed their decision, have chosen to give me access to development versions of the app, allowing me to work and debug much faster compared to the protected-app workflow.

### Update Check Bypass

To permanently disable the forced update mechanism and prevent the application from hanging on the splash screen, I modified the version comparison logic within `UpgradeManager$tryUpgrade$2.smali`. The original code compared the server version integer against the local version using a conditional branch (`if-le`). I patched this instruction by replacing it with an unconditional jump (`goto`), forcing the execution flow to always bypass the "Update Available" logic. This hardwires the application to execute the "No Update Needed" callback path every time, allowing the splash screen to dismiss normally regardless of the actual server version.
This allows to run any version of the app without worrying about updates.

### Unlocking Incomplete Games (LOOI Pong)

The app resources contain a fully functional activity `LooiPingPongActivity` (Pong), but it is hidden from the main menu's `GameCenterActivity`. Consider this game is broken and incomplete, and is literally just 2 player pong. Probably scrapped due to the (apparently) low performance 2 face recognition system.

**The Mod:**

* **Target:** `com.TangibleFuture.looiRobot.game.activity.GameCenterActivity`.
* **Menu Injection:** I increased the `GameBean` array size in `initData` from 3 to 4 and injected a new game item.
* **Logic:** The launch function `jumpGame` contains a check for a specific localized string. I hardcoded the menu item name to **"LOOI Pong"** (as there are some weird function limits I'm not confident with yet) and patched `jumpGame` to accept this string as the "password" to launch the hidden activity.

### RC Unleashed

The stock application artificially restricts Remote Control usage by monitoring the `phone_stick_state` variable (containing the magnetic attachment state). If the phone is detected as "Docked" on the robot, the app blocks the RC interface with a mandatory "Take Down Phone" dialog, forcing you to decapitate the poor thing. Hell, what if you want to drive it with cardboard LOOI on?

**The Mod:**

* **Target:** `com.TangibleFuture.looiRobot.activity.RemoteControlActivity`.
* **Logic Bypass:** I identified the sensor check routine within the `onResume` lifecycle method and removed the conditional jump that triggers the blocking dialog. Additionally, the `showTakeDownPhoneDialog` method was neutralized to prevent any asynchronous triggers.
* **Result:** The RC interface now loads immediately regardless of the phone's physical position, enabling fully functional driving controls while something is mounted on the robot.

### About LLM REing

It is fully possible on 2.5.0 and newer (both stt method and rtc method) and managed to use my local model with a modded client and a partial custom server. More details on the SOOPERCHARGE Discord server.

---

## About LOOI's BLE protocol

LOOI operates on a sequence-based BLE protocol. It is not a simple direct-drive remote control; it uses a "Script Mode" logic where commands act as frames in an animation sequence.

### Connection Handshake

(Human written!)
Unfortunately, you cannot simply connect via NRF Connect and start sending commands immediately. The robot requires a handshake sequence initiated by the official app (which, from my knowledge, simply asks the robot to output its available components) - If this handshake is missing, LOOI will disconnect after a few seconds. Spoofing a fake motor base is also possible.

Connect via the official app first, background it (do not kill it), then connect from NRF connect. :-)

### The Command Interface (`fe00`)

The primary write characteristic is `fe00`. It accepts multi-byte strings to coordinate motors, LEDs, and screen animations.
The command structure is GENERALLY a **17-byte packet**.

**Reference Command (Rotate and turn led red - it's a custom command, you're welcome to try it):**
`00 07 00 FF 05 00 00 00 00 64 02 0A 96 02 14 00 02`

### Known Characteristics & UUIDs (Incomplete)

| UUID Prefix | Function | Notes |
| --- | --- | --- |
| **fe00** | **Command Interface** | Main write channel for animations and movement. |
| **ff02** | **Motor Drive (Boost)** | High-speed movement. Sending `FF` supposedly changes direction. |
| **fed0** | **Motor Drive (Slow)** | Precision movement control. |
| **fed1** | **Neck Motor** | Independent head articulation. |
| **fed2** | **Headlight/Torch** | Direct control: `00` = Off, `03` = On. |
| **fed9** | **Sensor Data** | Read-only stream (Cliff sensors, TOF distance, Battery). |

---

## Contributing & Credits

**Big thanks to:**

* **u/revned911 (Reddit):** For initial findings on the BLE topic.
* **u/CunningLogic (Reddit):** For their research and unpacking work on DRM-protected (Jiagu360) versions of the app.
* **TangibleFuture** for providing development versions of the App useful for my work.

**Want to contribute?**
This project is active and I am looking for help! If you have experience with:

* Android Reverse Engineering (Smali/Kotlin)
* Bluetooth Low Energy packet analysis
* LLM Live Chat stuff

Please contact me personally on **Telegram: @SplattyDoesStuff**. I am looking forward to working with anyone who has more competence than me, especially on the code side!
Again, write AI messages to me and I will ignore you.
Write in official channels and you’ll get punished.
Releases are outdated and will not be maintained for a long time.
