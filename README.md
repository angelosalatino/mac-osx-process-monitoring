# MacOSx Process Monitor

This tool is a small bash script that routinely runs to check if a particular process with a given name uses a lot of CPU. 

This script has been designed because of an annoying feature of one of the software I must use: Citrix SSO, which is the software we use to connect to the VPN of my university.

After few hours of use, the deamon process VPNExtensionX, brings its CPU usage to 100% without any apaprent reason. This makes the laptop becoming very hot and the fans start speeding up. The IT department cannot fix it, and apparently killing it and relaunching it seems the only solution. 

Sometimes, for some reasons, I do not realise the CPU usage has gone high. This monitor will help me to improve the life expentancy of my machine.


# Files

## Script

### New Version

Path ```~/watch_vpn_proc```
```bash
#!/usr/bin/env bash

MAXTIMES=4
TIMESFOUND=$(/bin/launchctl getenv VPNTIMESUSAGE)
read pct name thispid< <(top -l 2 -n 1 -F -o cpu -stats cpu,command,pid | tail -1)
if (( ${pct%.*} >= 50)) && [ "$name" = "VPNExtensionX" ]; then
  if [ "$TIMESFOUND" = "" ]; then TIMESFOUND=0; else TIMESFOUND=$((TIMESFOUND+1)); fi
  if (($TIMESFOUND >= $MAXTIMES)); then 
    osascript -e 'display notification "Process > 50%: '"$name"' ('"$pct"'%) (pid: '"$thispid"'), counts: '"$TIMESFOUND"'. Will kill it now. Bye bye." with title "High Usage of CPU"';
    kill -9 $pid
    TIMESFOUND=0
  else 
    osascript -e 'display notification "Process > 50%: '"$name"' ('"$pct"'%) (pid: '"$thispid"'), counts: '"$TIMESFOUND"' " with title "High Usage of CPU"'; 
  fi;
else 
  TIMESFOUND=0;
fi
/bin/launchctl setenv VPNTIMESUSAGE $TIMESFOUND
```

### Old Version

Path ```~/watch_vpn_proc```
```bash
#!/usr/bin/env bash

read pct name thispid< <(top -l 2 -n 1 -F -o cpu -stats cpu,command,pid | tail -1)
if (( ${pct%.*} >= 50)) && (("$name"=="VPNExtensionX")); then
  osascript -e 'display notification "Process > 50%: '"$name"' ('"$pct"'%) (pid: '"$thispid"')" with title "High Usage of CPU"'
fi
```

Inspired by [https://stackoverflow.com/questions/24129903/notifying-when-using-high-cpu-via-applescript-or-automator](https://stackoverflow.com/questions/24129903/notifying-when-using-high-cpu-via-applescript-or-automator)

## LaunchAgent

Path ```~/Library/LaunchAgents/watch.vpn.process.plist```
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
        <string>/Users/aas358/watch_vpn_proc</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
    <key>StartInterval</key>
    <integer>15</integer>
    <key>StandardErrorPath</key>
    <string>/Users/aas358/stderr.log</string>
    <key>StandardOutPath</key>
    <string>/Users/aas358/stdout.log</string>
</dict>
</plist>
```

In the absolute paths ```aas358``` is the username on my machine. Please change it accordingly.

# Commands

**Load the agent**

```launchctl load ~/Library/LaunchAgents/watch.vpn.process.plist```

**Unload the agent**

```launchctl unload ~/Library/LaunchAgents/watch.vpn.process.plist```

**Check loaded agents**

``` launchctl list | grep VPN```

**Check log**

```tail /var/log/system.log```


# Troubleshooting

## 

```
MMM DD HH:MM:SS MACHINE com.apple.xpc.launchd[1] (watch_vpn_proc[6764]): Service could not initialize: 19H1519: xpcproxy + 7195 [252][FB7D36FA-1F35-3A7A-B0FF-B1FAA70E4ED5]: 0x1e
MMM DD HH:MM:SS MACHINE com.apple.xpc.launchd[1] (watch_vpn_proc[6764]): Service exited with abnormal code: 78
```

I fixed it be putting absolute paths in ```~/Library/LaunchAgents/WatchVPN_proc.plist```
Before ```<string>~/watchvpn_proc</string>```, now ```<string>/Users/aas358/watchvpn_proc</string>```
