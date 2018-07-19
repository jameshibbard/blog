---
title: Moving Mail to Local Folders in Thunderbird
layout: post
permalink: /moving-mail-to-local-folders-in-thunderbird/
tags:
  - email
  - thunderbird
comments: false
---

Recently, when moving mail from the uni's mail server to Thunderbird's Local Folders, a whole bunch of messages got corrupted. As you can see from the screenshot, the corrupted messages don't have a subject, "from" is just a minus sign and the date is always the same.

![This is how the corrupt messages were displayed](https://res.cloudinary.com/hibbard/image/upload/v1528827957/corrupt_messages_in_thunderbird.png "This is how the corrupt messages were displayed")

I tried everything suggested in [Mozilla's Knowledge Base](http://kb.mozillazine.org/Disappearing_mail "This article describes what to do if some or all of your messages disappear") to restore the emails in question, but unfortunately to no avail. To make matters worse, we didn't notice that the messages had been corrupted until about a month after it had happened and our mail servers only have a backup of the last ten days.

## Workaround:

This is not an optimal solution, but will ensure that you don't lose any mail when copying messages to your PC.

- Select the messages you wish to move from the remote mail server to your local folders
- Right click on one of the selected messages
- Select _Copy To_ / _Kopieren in_
- Select _Local Folders_ / _Lokale Ordner_
- Select the folder to which you would like to move the mails. **This will copy the mail to your PC,but not delete the original on the server.**
- Once the mails have been copied, check your local folders to make sure they are there and are previewing correctly
- Delete the original mails from the mail server

![Dialogue for copying mails from an IMAP server to the Local Folders](https://res.cloudinary.com/hibbard/image/upload/v1528828115/thunderbird_copy_dialogue.png "Dialogue for copying mails from an IMAP server to the Local Folders")

I also made enquiries as to whether we were doing anything incorrectly, and if it would have been better to archive the mails instead. Judging by the answers that I received, it seems there is very little difference between Thunderbird's (rather rudimentary) archiving system and its "Local Folders".

For more robust archiving options in Thunderbird, see this article: [http://kb.mozillazine.org/Archiving_your_e-mail](http://kb.mozillazine.org/Archiving_your_e-mail "Archiving your e-mail in Thunderbird")

For more information on the size limits in Thunderbird, see this article: [http://kb.mozillazine.org/Limits_-_Thunderbird](http://kb.mozillazine.org/Limits_-_Thunderbird "This article describes known limits in Thunderbird")
