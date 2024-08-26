# Detailed Guide to Stop Alexa Bluetooth Announcements

This README provides detailed solutions to prevent Alexa devices from announcing Bluetooth connections for Windows, Linux, and macOS systems.

## Detailed Problem Description

Alexa devices are known to make annoying announcements when connecting or disconnecting from Bluetooth devices, particularly when a computer enters or exits sleep mode. These announcements can be especially problematic in various situations:

- Interrupting activities that require concentration
- Disturbing meetings or calls
- Waking people up during the night
- Interfering with listening to music or podcasts

The problem is caused by Alexa interpreting short interruptions in the Bluetooth connection as a new connection, leading to frequent and unnecessary announcements.

## Solution

The root cause of this issue is that Alexa devices enter a standby mode after approximately 10 minutes of inactivity. When they detect any sound or activity after this period, they interpret it as a new connection and make an announcement.
To prevent this, we can create a script that runs every 9 minutes, emitting an imperceptible sound. This keeps the Alexa device "active" and prevents it from entering standby mode. As a result, Alexa doesn't perceive subsequent sounds as new connections, eliminating the need for announcements.
The solutions provided below for Windows, Linux, and macOS all follow this principle: they set up a scheduled task that plays a very short, nearly inaudible sound every 9 minutes. This constant, minimal activity keeps the Bluetooth connection "alive" in Alexa's perception, effectively stopping the annoying announcements.

## Detailed Solutions

### Windows

1. **Open Task Scheduler:**
   - Press `Win + R`, type `taskschd.msc`, and press Enter.
   - Or search for "Task Scheduler" in the Start menu.

2. **Create a new task:**
   - In the right panel, click on "Create Basic Task".
   - Give the task a name, e.g., "Keep Alexa Connection".
   - Select "Run whether user is logged on or not" 

3. **Configure the schedule:**
   - Go to the "Triggers" tab and create a new one.
   - Choose "When I log on".
   - Set it to repeat every 9 minutes (you can write it in the select box) and the duration to "Indefinitely".

3. **Configure the action:**
   - Go to the "Actions" tab and create a new one.
   - Select "Start a program" as the action.
   - In the "Program/script" field, enter `powershell.exe`.
   - In the "Add arguments" field, enter the following script:

     ```powershell
     -Command "$player = New-Object System.Media.SoundPlayer; $player.SoundLocation = 'C:\Windows\Media\Windows Background.wav'; $player.Load(); $player.Play(); Start-Sleep -Milliseconds 50; $player.Stop();"
     ```

5. **Save:**
   - Click "OK" to save the changes.

This script will play a brief, nearly imperceptible sound every 9 minutes, preventing Alexa from disconnecting.

### Linux

1. **Install sox:**
   Open a terminal and type:
   ```bash
   sudo apt update
   sudo apt install sox
   ```

2. **Edit the crontab:**
   In the terminal, type:
   ```bash
   crontab -e
   ```
   If prompted, choose your preferred editor (nano is a good option for beginners).

3. **Add the following line:**
   ```
   */9 * * * * XDG_RUNTIME_DIR=/run/user/1000 /usr/bin/play -n synth 1 sin 20
   ```
   This line will run the command every 9 minutes.

4. **Save and exit:**
   - If you're using nano, press `Ctrl + X`, then `Y`, and finally `Enter`.
   - For other editors, use the appropriate save and exit command.

This command will play an inaudible 20Hz tone for 1 second every 9 minutes.

### macOS

1. **Create an Apple Script:**
   - Open "Automator" from the Launchpad or Applications folder.
   - Choose "New Document" and then "Application".
   - Search for "Run AppleScript" in the search bar and drag it to the workspace.
   - Replace the content with the following script:

     ```applescript
     on run
         do shell script "afplay /System/Library/Sounds/Funk.aiff"
         delay 0.1
         do shell script "killall afplay"
     end run
     ```

   - Save the application with a name like "KeepBluetooth" in an easy-to-remember location.

2. **Create a Launchd job:**
   - Open TextEdit and create a new file.
   - Paste the following content:

     ```xml
     <?xml version="1.0" encoding="UTF-8"?>
     <!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
     <plist version="1.0">
     <dict>
         <key>Label</key>
         <string>com.user.keepbluetooth</string>
         <key>ProgramArguments</key>
         <array>
             <string>/Applications/KeepBluetooth.app/Contents/MacOS/Application Stub</string>
         </array>
         <key>StartInterval</key>
         <integer>540</integer>
     </dict>
     </plist>
     ```

   - Replace the path in the `<string>` element with the actual path to your application.
   - Save the file as `com.user.keepbluetooth.plist` in the `~/Library/LaunchAgents/` folder.

3. **Load the Launchd job:**
   Open Terminal and type:
   ```bash
   launchctl load ~/Library/LaunchAgents/com.user.keepbluetooth.plist
   ```

This will run the script every 9 minutes (540 seconds).

## Troubleshooting

- **Windows:** If the task doesn't start, check the Windows Event logs for errors.
- **Linux:** Use `grep CRON /var/log/syslog` to check if the cron job is running.
- **macOS:** Check `console.app` for any errors related to your Launchd script.

## Contributing

If you have improvements or alternative solutions, feel free to contribute to this guide. You can do so in the following ways:

1. Open an "Issue" on GitHub to discuss potential changes.
2. Create a "Pull Request" with your proposed modifications.
3. Share your experience in the discussions to help other users.

## Additional Resources

- [Official Alexa Documentation](https://developer.amazon.com/en-US/docs/alexa/alexa-voice-service/bluetooth.html)
- [Alexa Community Forum](https://www.amazonforum.com/s/topic/0TO4P000000E60xWAC/echo-alexa)
- [Windows Task Scheduler Documentation](https://docs.microsoft.com/en-us/windows/win32/taskschd/task-scheduler-start-page)
- [Cron Guide for Linux](https://www.adminschoice.com/crontab-quick-reference)
- [Launchd Guide for macOS](https://www.launchd.info/)
