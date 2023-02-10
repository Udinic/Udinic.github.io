---
author: udinic
comments: true
date: 2011-11-21 18:27:41+00:00
layout: post
slug: errors-in-the-error-reporter
title: Errors in the Error Reporter
wordpress_id: 132
categories:
- Android
- Libraries
tags:
- ACRA
- Android
- Crash Report
- Error Report
- Exception Handler
---

I'm working a lot with open source libraries, some of them just give you a source, short explanation, and...good luck! The others are well documented and their developers are active. On this post I'm bringing a nice story about integrating an Error reporting mechanism to [Any.DO](http://any.do) . After doing some investigation, [ACRA](http://code.google.com/p/acra/) was found as the best solution. And yes...it's open sourced, and from the second kind I've presented here.


ACRA integrates as the default exception handler in the program, and reports to a Google spreadsheet (on Google Docs), Email or to any place you'd like. It also presents the user with a friendlier dialog when a crash occurs. In each report you can define the data to send from a wide variety of information, starting with the stack trace and the LogCat of the last 2 minutes prior to the crash and up to the system's dumpsys and the saved shared preferences. If the user doesn't have internet connection at the time of the crash, or the reporting process failed from some reason, the report is saved on the device and is sent later. And it's really easy to use!




Anyway, we decided to send the crash reports to a Google spreadsheet along to our DB, making it easy to query for statistical information about the users' crashes. In the spreadsheet we kept almost all the information we can get, and on the DB we kept only the most important info, like the stack trace and the version information. We  also saved the AcraReportId on our DB, which helped us track down the extended information in the spreadsheet (if needed..). THE PROBLEM WAS that we got over 20,000 users on the first 48 hours to our launch (Hooray!), and some of our debug reports (reports intentionally sent to check issues raised on our beta stage)  made our spreadsheet fatter than Apple's bank account. At some point all the reports sent to the spreadsheet got IOException-ed (very few requests were successful), and we couldn't even open the spreadsheet nor download it! "At least we have that information on our DB.." I thought, but what I didn't know was that ACRA didn't send the reports to our DB as well!! Hmmmm....now it began interesting...




We were blind. The users count is getting bigger and bigger, and we didn't know why our DB gets only 3-5 new reports a day (I know our app is perfect, but not that perfect!). Since ACRA is open sourced, finding the cause for the problem was pretty easy. I've looked at the code responsible to send the reports:



{% highlight java %}
    private static void sendCrashReport(Context context, CrashReportData errorContent) throws ReportSenderException {
        boolean sentAtLeastOnce = false;
        for (ReportSender sender : mReportSenders) {
            try {
                sender.send(errorContent);
                // If at least one sender worked, don't re-send the report
                // later.
                sentAtLeastOnce = true;
            } catch (ReportSenderException e) {
                if (!sentAtLeastOnce) {
                    throw e; // Don't log here because we aren't dealing with the Exception here.
                } else {
                    Log.w(LOG_TAG, "ReportSender of class " + sender.getClass().getName()
                            + " failed but other senders completed their task. ACRA will not send this report again.");
                }
            }
        }
    }

{% endhighlight %}

The problem was obvious, if the first sender throws an exception - the next ones on the array will be ignored! Since the Google spreadsheet sender is one of the internal report senders, it precedes any custom sender made (like our beloved DB report sender). The report will be sent again later, but as long as there's a problem with the spreadsheet - it's doomed to fail. According to the code here, if the order of the array was different, our report would've been sent, and ACRA will not send the report again to the spreadsheet, since it was successfully sent already through other report sender. I filed an [Issue](http://code.google.com/p/acra/issues/detail?id=92&sort=-id&colspec=ID%20Stars%20Type%20Status%20Priority%20Milestone%20Owner%20Summary) about that behavior on the ACRA's project site, suggesting to try and send the report through ALL the report senders, before giving up. Hope it'll be fixed in future versions.

In order to overcome such issue until it'll be fixed, I found a little trick to make my custom report sender be the first on the array. Since the array is maintained on the ErrorReporter class, and it's a singleton, I just called the ErrorReporter.getInstance().addReportSender() BEFORE calling the ACRA.init(), which also calls the addReportSender() with the default report senders, as the Google spreadsheet for example.

Problem solved! And now...we wait for all the users to update to the new version. My DB and I will be waiting for all those pending error reports, failed to be sent on the previous version, to flood our DB with might-be-irrelevant error reports. I'll go make some coffee...
