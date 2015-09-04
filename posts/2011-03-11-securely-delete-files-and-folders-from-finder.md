---
layout: post
title: "Securely delete files and folders from Finder"
date: 2011-03-11 18:10:26 -0400
comments: true
categories: howto
alias: /2011/03/securely-delete-files-and-folders-from.html
---

In the computer world, when you delete a file and empty the Trash or Recycle Bin, it's not really gone. This can be a good thing for when you accidentally delete something critical, or your hard drive crashes and you need to hire professionals to recover these files for you. These files are still recoverable because they're still on the disk, just that you no longer have access to them. However in some cases, you may want to delete a file permanently and ensure that it is unrecoverable. 

<!--more-->

Enter secure delete. In a nutshell, what this does is overwrites the file's contents several times to ensure that whatever data may be left over is completely blown away. The number of times depends on how paranoid you are. Some would argue that one overwrite is enough, while others insist that the more the better. Keep in mind of course that the more you overwrite, the longer it takes to fully delete the file. This can be problematic when deleting several really large files in that it can take hours for the deletion process to complete. 

Mac OS X has two features that allow you to do a secure delete on a file. The first is through the Trash. If you right click on it and then press the command key, the Empty Trash label turns into Secure Empty Trash. The second is by a utility accessible through the command line, called srm, which stands for secure remove. 

The problem with using the Trash for a secure delete is that it will do a secure delete on everything in the Trash. You may not want this, you may want only one or two files to be permanently deleted. Furthermore, if you have a lot of large files in your Trash, doing a secure delete can take much longer than you expect. The problem with srm is that you have to be comfortable with the command line, and most people aren't. 

A solution to this is to create a Service such that when you right click in a file or folder, an option to secure delete appears in the pop up menu. Here's how it's done.

1. Go into your Applications folder and launch Automator. When it starts up you'll be asked what kind of project you'd like to use. Select Services.

    ![](/images/2011-03-11/step1.png)

2. Set the Service receives selected option to files or folders.

    ![](/images/2011-03-11/step2.png)

3. On the left side of Automator, select Library > Files & Folders and on the panel to the right of that, select Get Selected Finder Items. Once you've selected it, drag it to the large panel on the right to add it to the workflow. 

    ![](/images/2011-03-11/step3.png)

4. We need to add another item to the workflow. Back on the left side of Automator, select Library > Utilities, and Run Shell Script. As before drag it to the workflow panel and it should appear right under Get Selected Finder Items.

    ![](/images/2011-03-11/step4.png)

5. Once you've added Run Shell Script, click on its Pass Input' option and select as arguments. You should see the text in the input box change to the following: 

```
do
    echo "$f"
done
```

Delete all that and replace it with the following:

```
${HOME}/secure.delete.log
```

This simply means run srm on the selected files and create a log file in the user's home folder. 

![](/images/2011-03-11/step5.png)

Finally, click on File > Save, and save it as Secure Delete.

Alright, now let's take it for a test spin. Look for some files you want to delete, select them, and then right click on them. The contextual menu will appear and at the bottom you'll see Secure Delete. If you click on Secure Delete, the service we created will call the srm command on each selected item and begin a secure delete procedure on each of them. If the files are large, you'll notice it takes a while, but it'll eventually finish.

If you're curious as to whether it worked or not, go to your home folder and you'll notice the secure.delete.log file. Go ahead and open this and you'll see some information that srm generated when it was running. 

That's it! If you want, you can even add a keyboard shortcut for it so that instead of right clicking to open the contextual menu and running Secure Delete, you could just hit a key combination and it'd run automatically. I'll leave that as an exercise to the reader.
