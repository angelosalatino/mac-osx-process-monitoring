# MacOSx Process Monitor

This tool is a small bash script that routinely runs to check if a particular process with a given name uses a lot of CPU. 

This script has been designed because of an annoying feature of one of the software I must use: Citrix SSO, which is the software we use to connect to the VPN of my university.

After few hours of use, the deamon process VPNExtensionX, brings its CPU usage to 100% without any apaprent reason. This makes the laptop becoming very hot and the fans start speeding up. The IT department cannot fix it, and apparently killing it and relaunching it seems the only solution. 

Sometimes, for some reasons, I do not realise the CPU usage has gone high. This monitor will help me to improve the life expentancy of my machine.


# Files

## Script

Path ```~/watchvpn_proc```
```bash
#!/usr/bin/env bash

read pct name thispid< <(top -l 2 -n 1 -F -o cpu -stats cpu,command,pid | tail -1)
if (( ${pct%.*} >= 50)) && (("$name"=="VPNExtensionX")); then
  osascript -e 'display notification "Process > 50%: '"$name"' ('"$pct"'%) (pid: '"$thispid"')" with title "High Usage of CPU"'
fi
```

Inspired by [https://stackoverflow.com/questions/24129903/notifying-when-using-high-cpu-via-applescript-or-automator](https://stackoverflow.com/questions/24129903/notifying-when-using-high-cpu-via-applescript-or-automator)

## LaunchAgent

Path ```~/Library/LaunchAgents/WatchVPN_proc.plist```
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>KeepAlive</key>
    <false/>
    <key>Label</key>
    <string>WatchVPN_proc</string>
    <key>ProgramArguments</key>
    <array>
        <string>~/watchvpn_proc</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>StartInterval</key>
    <integer>15</integer>
    <key>StandardErrorPath</key>
    <string>~/stderr.log</string>
    <key>StandardOutPath</key>
    <string>~/stdout.log</string>
</dict>
</plist>
```

# Commands

**Load the agent**

```launchctl load ~/Library/LaunchAgents/WatchVPN_proc.plist```

**Unload the agent**

```launchctl unload ~/Library/LaunchAgents/WatchVPN_proc.plist```

**Check loaded agents**

``` launchctl list | grep VPN```

**Check log**

```tail /var/log/system.log```