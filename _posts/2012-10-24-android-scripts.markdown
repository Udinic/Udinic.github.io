---
author: udinic
comments: true
date: 2012-10-24 18:03:02+00:00
layout: post
slug: android-scripts
title: Android Scripts
wordpress_id: 341
categories:
- Android
tags:
- activities stack
- activity
- activity stack viewer
- adb
- alarms
- Android
- android alarms dumper
- Any.DO
- batch
- batch scripts
- dumpsys
- python
- scripts
- shell
- sqlite
---

As a busy/lazy developer, I always try to automate repeating routines. Those sequence of actions are getting annoying after a few times and always make me gasp. That’s why some of my time is dedicated to writing small scripts to make my life easier, and the development process faster and anger-free. Most of them are very specific to the apps I’m working on, and some are more generic. I’ve pushed them to a [new repository](https://github.com/Udinic/AndroidScripts) on GitHub, and it will updated with more scripts when I’ll make some more, or generalize other existing scripts that I use.


### ADB Batch Scripts


ADB – Android Debug Bridge, is a very powerful tool. You can control a lot of features on the phone. Furthermore, granting yourself with a ROOT access will bring many rewards, as getting files in and out the file-system, manipulating system files and more. I _highly_ recommend rooting your development phone; it’ll prove itself worthy towards the trouble that it might cause in some devices. I’ll also note that the root access from ADB is limited on production builds (stock ROMs) vs. custom ROMs (e.g. Cyanogenmod), so I prefer to just flash a custom ROM on my development phone, making me the master of its domain!

Most of my repeated ADB commands are for DB access. I need to check the app’s SQLite DB to see that my code did everything right. So I wrote this windows batch script:


{% highlight bash %}
adb root
adb pull /data/data/<package name>/<db filename> c:\temp
if %errorlevel% neq 0 goto :error
start d:\programs\SQLite_browser\SQLiteDatabaseBrowser2.0b1.exe c:\temp\<db filename>
goto :EOF

:error
echo Error accessing the DB
pause
exit /b %errorlevel%

:EOF
{% endhighlight %}




First, I request to start ADB with root access, not applicable on stock ROMs (as least not easily), and pulling out the DB file to a temp location. The first time running this will show an error, since the _adb root _command restarts the ADB daemon on you computer, which causes the next ADB command not to find the device momentarily. The second time running this will be the charm. To open the DB I use [SQLite Browser](http://sourceforge.net/projects/sqlitebrowser/), which is very simple and lite. Running SQL on the DB is buggy, but for simple table viewing – it’s good enough.

If we want to modify the DB on the device, I wrote a small script to receive any SQL statement, and run it directly on a DB inside the device:


{% highlight bash %}
adb shell sqlite3 /data/data/<package name>/<db filename> %1
{% endhighlight %}




Very simple. Using the sqlite3 command, we can get a SQLite prompt that’s running on the device itself. If you don’t have the sqlite3 file in your device, which can happen on stock ROMs mostly, you can use [this nice article](http://othell.com/wp/?p=48) that helped me do it on my Galaxy Nexus, or just search Google.

Another common thing I do, is executing specific services and activities, great help for my debugging life. The syntax is simple, but here’s an example to show how easy that it:


{% highlight bash %}
adb shell am startservice -n <package name>/<Service full class name> [<extras>]
adb shell am start -n <package name>/<Activity full class name> -a android.intent.action.MAIN [<extras>]
{% endhighlight %}




I can pass extras, action and even activity attributes as clear-top, no-animation and more. I recommend reviewing all the options for those commands, you might get ideas how to automate some of your routines.

Some tests require installing/uninstalling the app to clear the data fast or to test upgrading code. The _adb install _and _adb uninstall _will do the trick here. Very simple to use and fast. I usually install the newest version of [Any.DO](http://any.do) on my device using an ADB command, rather than from Google Play. It’s just faster!


### Python Scripts


Using Python, I could write more complex stuff, like analyzing a system dump, pulling only the important information. If you’re familiar with ADB, you probably heard of “_adb shell dumpsys”_. This command will print a dump of the system, containing LOTS of data about memory allocations, registered receivers, apps permissions and more. You can call “_adb shell dumpsys \<service name\>”_ to get a specific service’s information. Some examples:


{% highlight bash %}
# All the information about accounts on the device
adb shell dumpsys account 

# The battery state
adb shell dumpsys battery

# All the registered alarms on the device
adb shell dumpsys alarm
{% endhighlight %}




The problem with the last command – it prints out too much garbage. There are lots of alarms registered on your device at any given moment, and the dump’s printing format is not exactly an eye candy. Since there’s a lot of alarm handling on Any.DO, I needed a clear and easy way to look only at the relevant information. That’s why I wrote this script:


{% highlight python %}
p=subprocess.Popen([r'adb', 'shell', 'dumpsys alarm'],
                       stdout=PIPE, stderr=PIPE,
                       shell= True,
                       cwd='C:\\')
out, err = p.communicate()
p.wait()

pp = pprint.PrettyPrinter(indent=4)

lines= out.split('\n')
alerts = [["RTC TYPE", "When", "Repeat", "pkg", "Type"]]

for i in xrange(len(lines)):
    if lines[i].replace(" ","")[:3]=="RTC":

        # Extract the information we
        rtc_type = lines[i].strip().split()[0]
        when=re.findall('when=([0-9a-z+]*)',lines[i+1])[0]
        repeat=re.findall('repeatInterval=([0-9a-z+])*',lines[i+1])[0]
        intentType=list(re.findall("PendingIntentRecord{(\w+) ([\w\.]+) (\w+)", lines[i+2])[0])

        # If the user has passed a package name as an argument
        # we'll filter all other packages' alarms
        if len(sys.argv) > 1:
            if intentType[1] == sys.argv[1]:
                alerts.append([rtc_type, when, repeat, intentType[1], intentType[2]])
        else:
            alerts.append([rtc_type, when, repeat, intentType[1], intentType[2]])

out_lines = [""] * len(alerts)

# Format the data nicely to columns
for column in xrange(len(alerts[0])):

    for alert_index in xrange(len(alerts)):
        out_lines[alert_index] += str(alerts[alert_index][column])

    line_size = max(map(len, out_lines)) + 5

    for alert_index in xrange(len(alerts)):
        out_lines[alert_index] += " " * (line_size - len(out_lines[alert_index]))

for l in out_lines:
    print l
{% endhighlight %}




All I do here, is getting the output from the command “_adb dumpsys alarms”_, and extract the information I need the most:

1. [RTC](http://developer.android.com/reference/android/app/AlarmManager.html#RTC) type.
2. When will the alarm go off.
3. The repeat interval of the alarm, if there is one.
4. The intent type that the alarm will invoke. Could be a Service, Activity or a Broadcast.

Here’s an example output:

[![image](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image_thumb3.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image3.png)

Passing the package name as an argument will print only its relevant alarms.

Another issue I encountered was activities’ stack handling. All those activities’ attributes, “singleTop”, “singleInstance” and more, can get you utterly confused about what would happen when you press the back button. Will you be navigated to the previous activity? will you be thrown to your main activity? or is it to the launcher screen? [Reading the documentation](http://http://developer.android.com/guide/components/tasks-and-back-stack.html) will help you understand what **should **be, and using my “Activity stack viewer” will help you see what **is **happening back there. This script is parsing the dumpsys, as the previous script did, and prints the activities’ stack grouped by their package name:

[![image](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image_thumb4.png)](/assets/images/{{ page.date | date: "%F" }}-{{ page.slug }}/image4.png)

Here I can see that on the Twitter app, I opened the post activity from the home activity. I see all the other paused activities that haven’t been destroyed yet by the operation system. Passing a package name as an argument will print a filtered list;

That’s all for now. I hope I helped at least some of you to do things a little bit more efficient. I’ll keep that [GitHub repository](https://github.com/Udinic/AndroidScripts) updated with new scripts I’ll write in the future. If you have a script to contribute – go ahead and submit a PR!
