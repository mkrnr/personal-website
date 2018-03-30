---
layout: post
title:  "How to Disable Win+L Lock Shortcut While Keeping Lock Functionality"
date:   2018-03-30 19:00:00 +0200
categories: windows
---

**The problem:** When disabling the `Win+L` shortcut on Windows 10 using a registry entry, not only does the shortcut get removed but the lock functionality in general --- even when suspending the computer or putting it in hibernate.

**The solution:** Creating two scheduled tasks --- `disableLock` and `enableLock` --- with the Windows Task Scheduler to update the registry entry.
The goal is to disable the locking after unlocking the system to get rid of the `Win+L` shortcut and to enable the locking right before the system is about to get locked, suspended, or put into hibernate with a script.

**The downside:** The `Sleep` and `Hibernate` buttons in the windows menu will still not lock your computer and there will be no `Lock` button.
The locking will only be triggered when executing the lock/suspend/hibernate scripts.

If this sounds good to you, you can follow the instructions below.
The idea to use the Windows Task Scheduler to prevent the [UAC](https://en.wikipedia.org/wiki/User_Account_Control) prompt that usually is displayed when running `.reg` files is taken from [this tutorial by Shawn on tenforms.com](https://www.tenforums.com/tutorials/57690-create-elevated-shortcut-without-uac-prompt-windows-10-a.html). 
Another solution would be to disable UAC altogether but I didn't want to go that far.

First, we create the `disableLock` task.  

For this, open the Task Scheduler, for example by typing `taskschd.msc` in the `Run` dialog.  
Next, click on `Create Task...` in the `Actions` menu on the right.  

On the `General` tab, add the name `disableLock`, check `Run with highest privileges`, and select `Configure for: Windows 10`:

![disableLock General]({{ site.media_url }}/posts/2018-03-30-disable-general.png)

On the `Triggers` tab, add a trigger for `On workstation unlock`:

![disableLock Triggers]({{ site.media_url }}/posts/2018-03-30-disable-triggers.png)

On the `Actions` tab, add an action `Start a program` which runs the program `C:\Windows\regedit.exe` with the arguments `/S "<PATH TO>\disableLockWorkstation.reg"`:

![disableLock Actions]({{ site.media_url }}/posts/2018-03-30-disable-actions.png)

You can find `disableLockWorkstation.reg` [on GitHub](https://github.com/mkrnr/dotfiles/blob/master/windows/lock/disableLockWorkstation.reg).

On the `Conditions` tab, uncheck `Start the task only if the comupter is on AC power`:

![disableLock Conditions]({{ site.media_url }}/posts/2018-03-30-disable-conditions.png)

The `Settings` tab doesn't require changes.


Next, create another task with `Create Task...`.

On `General`, add the name `enableLock`, again check `Run with highest privileges`, and select `Configure for: Windows 10`:

![enableLock Conditions]({{ site.media_url }}/posts/2018-03-30-enable-general.png)

For `enableLock`, no trigger will be added.
Instead, we will later trigger the task using a script that will also trigger the locking/suspension/hibernate.

On `Actions`, add an action `Start a program` which runs the program `C:\Windows\regedit.exe` with the arguments `/S "<PATH TO>\enableLockWorkstation.reg"`:

![enableLock Actions]({{ site.media_url }}/posts/2018-03-30-enable-actions.png)

You can find `enableLockWorkstation.reg` [on GitHub](https://github.com/mkrnr/dotfiles/blob/master/windows/lock/enableLockWorkstation.reg).

On the `Conditions` tab, again uncheck `Start the task only if the comupter is on AC power`:

![enableLock Conditions]({{ site.media_url }}/posts/2018-03-30-enable-conditions.png)

Again, the `Settings` tab doesn't require changes.


As mentioned above, the `enableLock` task will be triggered via scripts.
I decided to use Powershell scripts since they provide a way to run both suspend and hiberate without any additional tricks on Windows 10.
The `enableLock` task is triggered with `schtasks /run /tn "enableLock"`.
You can find my scripts `lock.ps1`, `suspend.ps1`, and `hibernate.ps1` [on GitHub](https://github.com/mkrnr/dotfiles/tree/master/windows/lock).

The very last step is to create shortcuts for the start menu to easily trigger the scripts.

For this, open `C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Start Menu` in the File Explorer and create a new shortcut with the name `Lock` (or similar).  
Open `Properties` and on the `Shortcut` tab, set the target `%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe -noprofile -executionpolicy bypass -file "<PATH TO>\lock.ps1"`.  
Repeat this to add shortcuts for the `suspend.ps1` and `hibernate.ps1` scripts.

Let me know if you run into any issues or have remarks about this approach.
