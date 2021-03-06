# How to transfer your email (with labels!) from one Gmail account to another
#### *Mac Edition*
For the obsessive among us who carefully curate their emails into
different folders (which in Gmail are called "labels"), there is no straightforward
online guide for transferring emails with their associated labels/folders from one
account to another.

If you're comfortable doing basic things on the terminal, then this is
your guide. Note that this guide is for Mac users. For the Windows guide, go
[here](./gmail-transfer-guide-windows.md).

## 1. Download and set up `got-your-back`
There's a command line tool with a cute name (`got-your-back`) on GitHub that
directly calls Gmail's APIs, doing all the heavy lifting needed to transfer
emails and labels. The process involves first creating a local backup of those
emails and labels from one account and then restoring them from this backup onto
another account.

Download the latest release of `got-your-back` from this
[link](https://github.com/jay0lee/got-your-back/releases).
It should look something like `gyb-1.0-macos.tar.xz`.

Open terminal.app, and&#8212;starting with `cd`&#8212;type the following to
change to the directory that contains the download, hitting enter to run the
command:
```
$ cd ~/Downloads
```
Unzip the download with `tar`, and then enter the unzipped directory:
```
$ tar -zxf gyb-1.0-macos.tar.xz
$ cd gyb
```

## 2. Backup
Suppose you wanted to transfer emails and labels from `old.email@gmail.com`
to `new.email@gmail.com`. This would take just two lines on the terminal.

First, to back up:
```
$ ./gyb --email old.email@gmail.com --action backup --local-folder my_backup
```

A prompt will appear looking like this:
```
Select the actions you wish GYB to be able to perform for old.email@gmail.com

[ ]  0)  Gmail Backup And Restore - read/write mailbox access
[ ]  1)  Gmail Backup Only - read-only mailbox access
[ ]  2)  Gmail Restore Only - write-only mailbox access and label management
[*]  3)  Gmail Full Access - read/write mailbox access and message purge
[ ]  4)  No Gmail Access

[*]  5)  Groups Restore - write to Google Apps Groups Archive
[*]  6)  Storage Quota - Drive app config scope used for --action quota

      7)  Continue
```

Press `7` and then the enter key, and a prompt will appear in your browser.
Enter your credentials and press `Allow`. Then you should see a
confirmation screen which says `The authentication flow has completed.`.

Looking back at the terminal, the backup process should have begun:
```
Using backup folder my_backup
Got 22648 Message IDs
GYB needs to examine 22648 messages
GYB already has a backup of 0 messages
GYB needs to backup 22648 messages
backed up 900 of 22648 messages
```
> Note: the number `22648` will differ depending upon the number of emails you
> need to back up.

Now go and make some tea. This process can take over an hour, if you have more
than 3 GB of emails. But when the prompt is done, your emails will be all backed
up, with the contents of the backup in a local directory called `my_backup`
located in your downloads folder at `~/Downloads/gyb/my_backup`.

## 3. Restore
You're now going to take your local backup and restore it to your new email,
`new.email@gmail.com`, effectively transferring the contents of one
account to another.

To restore, type the following:
```
$ ./gyb --email new.email@gmail.com --action restore --local-folder my_backup
```

As with `old.email@gmail.com`, you will once again see a prompt with options
1-6, and you will once again press `7` and then enter. As before, this will bring
up the browser. Enter your credentials and click `Allow` to bring up the restore
process. You will now see the following in the terminal:

```
Using backup folder my_backup
restoring single large message (9/6490)
```

`got-your-back` will restore messages 10 at a time, or 1 at a time if the messages
are very large. I've found that this process can take up to 5 times longer than
the backup, so be sure to modify your machine's energy saver settings to prevent it
from falling asleep, and then go and run some errands. I left mine working overnight.

If you go to `new.email@gmail.com`, you should see your old labels appearing, a
sign that the magic is working!

You can close the terminal once it's done restoring. The last step is
entirely in the browser!

As a cleanup step, don't forget to delete, `gyb-1.0-macos.tar.xz` and `gyb`, both
located in your downloads folder. They are no longer needed.

## 4. Set up forwarding
> Note: this last step is optional in the case that `old.email@gmail.com` continues
> to receive emails.

Open your old email account in Gmail and click on the settings ![][settings_button]
button. Click `Settings` and the tab called `Forwarding and POP/IMAP`. Then click
on the button that says `Add a forwarding address`.

A prompt will appear. Enter your `new.email@gmail.com` email address in the text box,
successively clicking `Next`, `Proceed`, and `Okay`. You may need to confirm the
forwarding in a message sent to `new.email@gmail.com`.

Go back to the settings in `old.email@gmail.com` under ![][settings_button] > `Settings` >
`Forwarding and POP/IMAP`. Under `Forwarding:`, click the radio button next to
`Forward a copy of incoming mail to`, make sure `new.email@gmail.com` is selected,
and in the third selector, choose whether you want email sent to your old account
to be kept, marked as read, archived, or deleted. Don't forget to click the `Save
Changes` button at the bottom of the page. 

And that's it!

[settings_button]: http://lh6.ggpht.com/snsP5-ODgFFqVJhxS5La7OAqsAmO-GwYWWERMFPW5R4MXcxp0zUZ5Bq6lRFqrvk92lA=w18-h18

## Additional Reading
Parts of this guide are graciously borrowed from the
`got-your-back` [Wiki](https://github.com/jay0lee/got-your-back/wiki) and Google's
[Gmail documentation](https://support.google.com/mail/answer/10957?hl=en), so please
feel free to go back to those resources for more detailed descriptions of any of
the steps described above.

## Troubleshooting

As a preface, there is a
[Google Group](https://groups.google.com/forum/#!forum/got-your-back) devoted to
`got-your-back` that you should definitely consult if you run into any issues.

Also note that by default, the backup and restore operations are idempotent. This
means that if you run the restore command again, for example, your email account
won’t have multiple copies of the same emails from the different restores, just a
single copy of them. In fact, rerunning backup or restore will take less time
because the tool won’t try to iterate over emails that have already been transferred.

### 'Error 400: Invalid From header'
```
restoring 10 messages (20847/22644)                                             
ERROR: 400: Invalid From header. Skipping message restore, you can retry later with --fast-restore
restoring 10 messages (20941/22644)                                             
ERROR: 400: Invalid From header. Skipping message restore, you can retry later with --fast-restore
restoring 1 messages (22644/22644)  
```
This warning appears if there's something weird about the message being restored.
This can either be that the message has an unusual label (for example, weird
characters or the label name is a reserved name) or, as I learned, the sender of
the message (the `From` field) is unusually formatted or too long.

There are two possible approaches:

1. Add the `--fast-restore` flag at the end of your `gyb` restore command. Note
that the messages restored this way will not be placed within their original
conversations after restore and possible duplicates of these messages will not be
removed. If only a handful of messages threw this error, then this flag might be
the way to go.
2. If you want to actually diagnose the problem, add the following string as a suffix
   to the restore command:

   ```
   --debug > debug.log
   ```
   This sends the full debug output to the `debug.log` file, which provides
   a detailed description of the messages being restored and the APIs called
   by the underlying Python script. From the `debug.log` file, you can
   identify the JSON objects that store the messages that couldn't get
   restored. Note that the `raw` field within those JSON objects encodes the
   actual message text using the `urlsafe_b64encode` method of the `base64`
   Python module.
    
