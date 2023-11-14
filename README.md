# How to sign your commits to GitHub with GPG

This document will discuss how to set up a Windows or macOS based toolchain that lets you push "verified" commits to your GitHub repositories by signing them with the help of a GPG key.

GitHub themselves provide a very elaborate and intricate [explanation in their Docs](https://docs.github.com/en/authentication/managing-commit-signature-verification) about how this works and how to set it all up, but while their descriptions are very precise and detailed, they still only provide a "how-to" but don't seem to concern themselves with "why" or "what".

The official Git SCM website offers a concise [description of the process](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work), but it restricts itself to doing everything on the command line, and it doesn't consider (visual) client-based work, also not interaction with an online-hosted repository system like GitHub.

I aim to make the topic understandable (to some extent) and less cryptic for people with little terminal and command line experience. (Though some keyboard interaction will be required.) After following these instructions, you should have a work environment that can sign your Git commits to GitHub repositories, and you should roughly understand what's going on in the background.

<br>

## Why sign Git your commits?

The simple answer is: to prove that it was really you who made the commit, and not someone impersonating you with a fake profile or spoofed Git credentials. Even if someone hacks or hijacks your GitHub account, they won't be able to sign and "verify" their malicious commits, at least not easily.

Also, GitHub will display a beautiful [green "Verified" badge](https://docs.github.com/assets/cb-17614/mw-1440/images/help/commits/verified-commit.webp) on a signed commit - and who doesn't love green verification badges!

If you care about more details, here are two good reads concerning the topic:
* https://dlorenc.medium.com/should-you-sign-git-commits-f068b07e1b1f
* https://withblue.ink/2020/05/17/how-and-why-to-sign-git-commits.html

<br>

## Preface

Some graphical Git clients (e.g. previously [GitHub Desktop](https://desktop.github.com/), or currently [SourceTree](https://www.sourcetreeapp.com/)) come with their own embedded Git installations, these are only available to the client app but not to  the operating system nor any other installed applications. This is done to avoid the typical "I installed it but it doesn't work" pitfall of missing dependencies, and to avoid potential Git version conflicts.

This document will focus on enabling GPG signing for the Git installation available to the operating system and other applications, **not** any such client-specific embedded Git versions. With the system-wide Git and GPG installations correctly set up and configured, GPG signed commits can be made from any client application on your system, no matter if command line or visual application. If you want to work with Git in the terminal, or with a visual Git client that uses the system-wide Git installation (e.g. GitHub Desktop), then this is the goal you want to reach.

If you use a visual Git client that **does** come with its own embedded Git installation (e.g. SourceTree), then depending on the client software, you might be able to switch your Git client over to use the system-wide Git instead of its embedded version. SourceTree has a setting for this, other clients may not. *(If your Git client can not be configured to use the system-wide Git installation instead of its embedded version, then the process described below will not work.)*

This document assumes that your machine has nothing related to this topic installed yet, no Git, no GPG and no GPG keys. Feel free to skip over any sections that explain things you're already aware of, or that describe processes you've already dealt with yourself.

This document assumes that you already have a GitHub account. If you haven't created a GitHub account yet, you can [do so here](https://github.com/join).

This document assumes that you already have a visual Git client installed, or that you are comfortable using command line Git in a terminal window. If you know nothing about command line Git, and you don't have a visual Git client installed yet, I would recommend to start with [GitHub Desktop](https://desktop.github.com/) (free) or [SourceTree](https://www.sourcetreeapp.com/) (free) first. Other free and commercial Git clients are available, but these two are a good starting point and they don't cost you anything. *(If you choose SourceTree, please note that you should switch it from the embedded Git version over to the system-wide Git version in its preferences.)*

<br>

## Sequence

[Prerequisite knowledge](#prerequisite-knowledge)<br>
Before everything else, I'll offer some explanations for the "terminal-ly illiterate", i.e. users who try to avoid a terminal or command line window at all cost. You won't be able to set this up without touching a terminal window, so I might as well try to give you a brief introduction that hopefully demystifies those black boxes a little bit.

[Install local Git version](#install-local-git-version)<br>
As a first step in the process, you'll set up a local Git installation on your machine. This Git installation needs to be accessible to your operating system and other applications.

[Install local GPG tools](#install-local-gpg-tools)<br>
You'll also need a system-wide installation of the GPG tools for your platform, this will provide ways to handle and maintain GPG keys. The GPG tools also need to be accessible to the system and other applications.

[Create GPG keys](#create-gpg-keys)<br>
With the GPG tools up and running, you'll create a pair of GPG keys.
- The **public** key will be the one you share with others, e.g. GitHub, to verify that something was signed by you.
- The **private** or **secret** key will be the one you keep safe and **don't give to anyone**! This is the one you use in your local toolchain to sign your Git commits. (You can import and use this on your other development machines as well.)

[Add public GPG key to GitHub](#add-public-gpg-key-to-github)<br>
Once you send commits to GitHub that were signed, GitHub will need a way to verify that your commits were correctly signed. You'll add your **public** GPG key to your GitHub account, so that GitHub knows to connect your commits to your account, and how to verify if they were correctly signed.

[Make local Git sign commits with your secret GPG key](#make-local-git-sign-commits-with-your-secret-gpg-key)<br>
Now that GitHub is prepared to handle incoming signed commits, and you have the GPG tools on your machine for Git to use, it's time to configure your local Git installation to actually use the GPG tools and start signing your commits.

<br>

## Prerequisite knowledge

### What do all these letters mean?!

In my instructions, you will come across similar but different acronyms like PGP and GPG. Don't get them confused.

- **PGP** stands for *Pretty Good Privacy*, an originally open encryption standard that got bought by McAfee and made closed-source, eventually purchased by Symantec.

- **OpenPGP** is an open encryption standard that was developed while the original PGP was closed-source and protected by McAfee.

- **GPG** is short for GnuPG is short for GNU Privacy Guard, the first open-source implementation of the OpenPGP standard, and an alternative to the closed-source PGP applications by McAfee and Symantec.

For more details, read through the [Wikipedia article about PGP](https://de.wikipedia.org/wiki/Pretty_Good_Privacy).

### What is a Terminal?

When I refer to a "terminal" in this document, I mean either the Terminal app (macOS) or a CMD.exe window (Windows). Many other terminal applications are available, e.g. Windows has PowerShell and Git comes with its own Git Bash. It's up to you which one you use, but in the scope of this document, I will assume it to be either Terminal or CMD.exe, simply because they're already pre-installed on most systems.

A terminal will give you access to the file system and operating system functionality, as well as potentially installed programs and scripts, all through a text-based interface rather than graphics and mouse clicks.

You type certain text commands into the window, then you press Enter to confirm your input and send off your command. Depending on the command you entered, you may or may not see responses from the program you ran output into the terminal window.

Although a terminal provides a text-based interface, it will usually not offer any advanced text editing capabilities. It is usually possible to highlight and copy text from the terminal window to the system clipboard, or to paste (with *Ctrl-V* on Windows or *Cmd-V* on macOS) text from the system clipboard into the command line. But cutting and pasting text fragments around on the current line of text is usually not possible.

### How to open a terminal window?

- On **macOS**, you can launch the Terminal app via Spotlight, just press *CMD+Space* on your keyboard and start typing "Terminal". Click the app entry once it becomes available in the search results, or select it with the Up/Down Arrow keys on your keyboard and launch it by confirming with Enter. Alternatively, you can launch it from Finder in *Applications/Utilities*.

- On **Windows**, click the Start icon (Windows logo) at the far end (usually left) of your task bar, then start typing "cmd" until the "Command Prompt" application becomes available in the search results. Once it's listed, right-click the "Command Prompt" entry and select "Run as Administrator" from the context menu. This will make sure you have administrative privileges, which should avoid a lot of potential issues revolving around insufficient permissions.

*Please note that this process MAY work in a non-administrator CMD prompt as well, but I didn't try it so I can't make any promises. If you don't have access to a Windows account with administrative privileges, and you get permission related issues following the steps outlined in this document, then please find someone who has an account with administrative privileges.*

### Basic terminal interaction

To show which folder the terminal is currently working in, run ```pwd``` (macOS) or ```cd``` (Windows) from the command line.

To change from one folder to another, both systems have the ```cd [TARGET]``` command, short for *Change Directory*.

Run ```cd ..``` to navigate into the parent folder.

Call ```cd``` followed by the name of a sub-folder in the current directory to have terminal change into that sub-folder, e.g. ```cd subfolder```.

You can also combine these two in order to directly navigate from the current folder into a sub-folder contained in any other parent folder, by doing something like this: ```cd ../../temp/log``` (macOS) or ```cd ..\..\temp\log``` (Windows). *(This would move up two parent folders, then navigate into the contained **temp** folder, and finally into the **log** folder inside that.)*

While the double dots ```..``` stand for the parent folder, the single dot ```.``` refers to the current folder, i.e. the current working directory of the terminal. You can always open the terminal's current working directory in your operating system's file browser by typing ```open .``` (macOS) or ```explorer .``` (Windows) and confirming with **Enter**. *(Note the space and dot after the commands.)*

An important caveat for navigating through folders has to do with spaces. If the file or path name contains a space, you will need to *escape* it: ```cd ../a\ folder\ with\ spaces/ ``` *(**Escaping** means to put a special symbol before the space, so the terminal knows to interprete it as part of a continuous string, not a delimiter to separate two command line arguments. This **Escape Character** is usually the ```\``` backslash.)*

Another fairly easy way to avoid any grief from spaces in file or folder names, and to keep your command line free of weird slash sequences, is to just put *""* quotes around any file or folder location, e.g. like this: ```cd "../a folder with spaces/"```.

Some terminal applications offer path and filename completion (or at least suggestions) if you tap the *TAB* key while typing a file system path or filename.

In some terminal applications, you can cycle through previously submitted commands with the *Up Arrow* and *Down Arrow* keys.

An **important key combination** to remember for terminal windows is *Ctrl-C*. If you entered a command, and the program you started is taking way longer than it should without signalling that it has stopped running (i.e. the terminal appears to be stuck), then you can usually terminate that program by pressing *Ctrl-C*.

If you've entered some text into the command line, but it's not what you want, and you just want to quickly discard the text without deleting every single character, also just press *Ctrl-C*.

<br>

## Install local Git version

Let's get Git out of the way first, simply because many \*nix based operating systems (like Linux and macOS) likely already have a version of Git installed.

### Check if Git is already installed

Check if a version of Git is already installed on your machine, and if it's available to the operating system as well as other applications. Open a terminal window and run this command:

```
git --version
```

If your terminal shows actual version information, something like ```git version 2.42.0.windows.2```, then you already have what you need and you can skip ahead to [the next section](#install-local-gpg-tools).

If your terminal spits out a complaint that it doesn't understand what "git" is, then you don't have a system-wide Git version installed.

If your machine does **not** seem to have a Git version installed and accessible from your terminal, you'll need to install one now.

### Installing Git with a package manager
 
If you know how to do so, you're welcome to install Git through your favourite package manager from a terminal. There's no reason I could think of not to do this, if you're already used to the process anyway.

For everyone else, there are simple platform-specific ways to get Git up and running on your machine:

### Installing Git on Windows

On **Windows**, just download the latest installer from [GitForWindows](https://gitforwindows.org/) and step through the setup process.

### Installing Git on macOS

On **macOS**, find [Xcode on the Mac App Store](https://apps.apple.com/us/app/xcode/id497799835) and install it. Xcode is a huge development environment for just about anything that has to do with Macs, it's provided by Apple for free, so you won't have to pay to use it - but it takes ages to download and (automatically) install, due to its significant size.

After Xcode is done installing, launch it once and give it a few seconds to let it get settled in, then open a terminal window and run the following command:

```
xcode-select --install
```

This will trigger a bunch of things to download and install in the background, and it may take quite a while. Once all the downloading and installing has finished, quit out of the Xcode app again. (You won't need to use Xcode for the activities outlined in this document, only some of the command line development tools it conveniently installs.)

### Check if Git is installed now

Check again if the new Git is correctly installed and available to your system and applications. Run the same command as before in a terminal window:

```
git --version
```

If your terminal shows actual version information, the installation was successful.

But if your terminal spits out a complaint that it doesn't understand what "git" is, then you still don't have a system-wide Git version available, something might have gone wrong during the installation. *(Troubleshooting potential installation issues exceeds the scope of this document, but try searching any error messages with your preferred search engine page.)*

<br>

## Install local GPG tools

With Git handled, let's focus on GPG. The GPG tools consist of several components, both command line applications and features that work in terminal windows, as well as a set of GUI applications to make use of them. While the command line applications work and behave the same on all platforms, the GUI applications are a little different.

This document will explain how to use all three alternatives to create and handle your GPG keys: the command line way, the Windows GUI application way, and the macOS GUI app way.

### Check if GPG is already installed

Just like with Git, check first if the GPG tools are already installed and available on your system by opening a terminal window and running the following command:

```
gpg --version
```

If you see a bunch of text about versions and supported algorithms, then you already have the GPG tools installed and you can probably skip ahead to [the next section](#create-gpg-keys).

If your terminal window complains that it doesn't understand the "gpg" command, then follow the steps below to install GPG now.

### Installing GPG tools

Download and run the GPG tools installer appropriate for your platform:
- macOS: [GPG Suite download](https://gpgtools.org/)
- Windows: [GPG4Win download](https://www.gpg4win.org/)

On **Windows**, the installer will ask you to specify an installation path. If you don't install GPG4Win to the default path at ```C:\Program Files (x86)\GnuPG\\```, then please **make a note of the installation folder**. You'll need to know the path to the GPG executable file later.

### Check if GPG is installed now

To verify if the GPG tools were correctly installed, run the following command in a terminal window.

```
gpg --version
```

If your terminal displays actual information about GPG's version and supported algorithms, the installation was successful.

If your terminal spits out a complaint that it doesn't understand what "gpg" is, then you still don't have the GPG tools installed correctly, something might have gone wrong during the installation. *(Troubleshooting potential installation issues exceeds the scope of this document, but try searching any error messages with your preferred search engine page.)*

<br>

## Create GPG keys

*This section assumes you don't already have any GPG keys. If you already have a valid set of public and secret GPG keys, then you may skip ahead [to the next section](#add-public-gpg-key-to-github).*

Let's create a pair of GPG keys.
- The **public** key will be the one you distribute to others, like GitHub later.
- The **secret** key will be the one you stash away and don't tell anyone about, this is the one you need to sign your commits (or encrypt files, emails, etc.) with.

In contrast to some other certification or verification methods, GPG keys are kept **per user** and not per machine. So if you code on several different computers using the same identity, it's sufficient to create one set of GPG keys and just export/import them between your computers. *(Just make sure you don't leave behind your **secret** GPG key on any public or shared computers!)*

When generating a key pair, GPG will ask you for your name and your email address. You can enter a fake name if you wish, but the name\* and email address must match the ones you use to make your commits. *Your name and email address stay on your computer, GPG doesn't send them away over the Internet, they are merely used locally to generate your GPG keys.*

(\* In all honesty, I don't know if the name you use with GPG has to match e.g. your GitHub user name. But for the sake of simplicity and credential organization, let's assume it does.)

**IMPORTANT** - If your GitHub profile is configured to "Keep my email address private" and to "Block command line pushes that expose my email", then you will not be able to push any commits made with your regular GitHub email address. GitHub will expect the commits to be made with a special no-reply email address, something in the format of ```USERNAME@users.noreply.github.com```.

You can check these settings and your special no-reply email address on the [Emails page](https://github.com/settings/emails) of your GitHub account. If your GitHub account expects such a no-reply email address, then THIS is the email address to associate with your GPG key pair, NOT your regular email address.

Once created, your GPG keys will automatically become available to the system. To move your GPG keys between computers, and to get e.g. your **public** key into GitHub, you'll need to *export* them from the system into files, I will tell you how later.

The recommended format for exported GPG key files is called *armored*, which is just a fancy name for a Base64 encoded plaintext representation of binary information that could otherwise not as easily be handled as text. The common file extension for *armored* files is ```.asc```, but for ease of use it's totally fine to use the ```.txt``` extension instead, that's up to you. *(Note that some GPG GUI applications may not like using extensions other than the ones commonly associated with GPG keys, like ```.asc```.)*

The default encryption method (at the time of writing) for GPG keys is "RSA and RSA" with a key length of 4096 Bits. You don't **have** to use exactly these settings if there's any reason not to, GitHub supports [a number of different encryption methods](https://docs.github.com/en/authentication/managing-commit-signature-verification/checking-for-existing-gpg-keys#supported-gpg-key-algorithms), but in this document I will stick to the standard values for simplicity.

A pair of GPG keys can be assigned an *expiration date*, which basically means that after a specified date, the GPG keys can't be used anymore. This is a worthwhile security measure, as anyone who might get hold of your **private** key will not be able to use it maliciously after the expiration date. On the other hand, constantly updating your keys on several systems can be a veritable nuisance, so you may want to opt against using an expiry date altogether. I'll leave the decision whether to use an expiry date or not up to you.

Your **secret** GPG key, i.e. the one that will be used to sign your commits, can (and should) be protected with a password. The safer option is definitely to use a password, but I'll leave it up to you to decide whether you really want to use a password or not, you don't have to. Just let me mention that every time the **secret** GPG key is used to sign something, i.e. every time you commit somthing with Git, you will be asked to enter this password. So if you do opt for a password (which you should), make sure that you can easily remember and type it, and/or that you store it in your password manager for easy access when needed. Now, don't tell me you still don't use a password manager...

Let's look at the three ways to create a new GPG key pair:
- [The terminal way](#the-terminal-way)
- [The macOS GUI way](#the-macos-gui-way)
- [The Windows GUI way](#the-windows-gui-way)

<br>

### The terminal way

Open a terminal window and enter the following command:
```
gpg --full-generate-key
```

This will cause GPG to ask you a few questions by printing a list of options to the terminal. It expects you to answer the questions by typing something and confirming your answer with *Enter*.

When GPG asks you which encryption method to use, take note of the number to the left of the option that says **RSA and RSA**. Type this number as your answer and confirm with *Enter*.

GPG will next prompt you for the key size, enter the number ```4096``` and confirm with *Enter*.

If you don't want the key to use an expiration date, type the number ```0``` when prompted and confirm with *Enter*.

If you **do** wish to assign an expiration date to your GPG keys, then type a number and a letter and confirm with *Enter*. The letter ```w``` means "weeks", the letter ```m``` means "months", the letter ```y``` means "years". No letter means "days". So for an expiry date in 3 days, you'd just enter ```3```. For an expiry date in 3 years, you'd enter ```3y```.

(After entering an expiration date (or not), GPG might ask you "is this correct?" Either confirm by typing the letter ```y``` followed by *Enter*, or return to the expiration date prompt by typing the letter ```n``` followed by *Enter*.)

As mentioned before, GPG will ask you for your "real name", but you don't **have** to provide your real name - unless there is a reason to, e.g. when using the GPG key pair for official email communication etc. When you're prompted, just enter the name you want and confirm with *Enter*.

When asked for an email address, enter the one associated with the identity you're going to use when pushing commits to your GitHub repository. As pointed out before, this could be either your regular account email address or a special GitHub no-reply email address, it all depends on how your GitHub account is configured. Type in the email address, confirm with *Enter*.

If you wish to add a comment to your key pair, e.g. to mention a special use for it or something, then do so when prompted, as usual confirm with *Enter*. Comments are not required, you could just press *Enter* without typing any text to skip.

At this point, GPG should present you with a summary of what you entered so far. If everything looks good and you wish to proceed, type the letter ```o``` and confirm with *Enter*. If you wish to make edits to the **N**ame, **C**omment or **E**mail address, then type a letter ```n```, ```c``` or ```e``` accordingly and confirm with *Enter*. Make your changes, return to the summary and confirm with the letter ```o``` and *Enter* when ready.

GPG will ask you to tap a few keys on your keyboard and erratically move your mouse around the screen for a few seconds. This creates "irrational chaos" and helps to improve the strength and "unbreakability" of the encryption for your GPG key pair.

Once GPG has collected sufficient random information, it will ask you to enter the password to protect your **secret** GPG key with. As mentioned before, a password isn't strictly necessary, so you could just confirm an empty password, but it would be advisable to use one.

*Should you decide not to enter a password to protect your **secret** GPG key with, you may have to go through the rigmarole of "enter a password" and "press keys and move mouse" two or three times before GPG will finally accept your decision. Just repeat your input and choice.*

If you now see a few lines of cryptic numbers and words printed to the terminal, as well as the name and email address you specified, then it means generating your GPG key pair was successful. Yay!

Now, let's figure out the exact ID of your GPG key, you might need this ID later to perform further actions on your key pair. Run the following command in your terminal: ```gpg --list-secret-keys --keyid-format=long```

This will print a list of all GPG keys currently registered on your system. If you see more than one entry listed, locate the one with the name and email address matching the ones you specified in the previous steps.

The first line of the entry should be in a format like ```sec   rsa4096/[RANDOM_LETTERS_AND_NUMBERS] [DATE] [LETTERS]```. The GPG key ID we're after is the *RANDOM_LETTERS_AND_NUMBERS* part. Click and drag over those letters and numbers to highlight/select them in your terminal, then press *CMD-C* (macOS) or simply right-click (Windows CMD) to copy them to your system clipboard. Paste the key into a text editor, and (for now) save the file somewhere as ```gpg_key_id.txt```.

The final step for this section will be to export your **public** and **secret** GPG keys into files. As mentioned before, I will use the file extension ```.txt``` here for ease of use, but you're welcome to use the standard ```.asc``` file extension for armored files instead. The actual file names don't matter, but I'll go with ```gpg_key_public``` and ```gpg_key_secret``` here to keep it simple and obvious.

First, let's get your **public** GPG key into a file named ```gpg_key_public.txt``` in (and this is important for finding the file later) **the directory your terminal is currently operating in**. *(Remember the ```open .``` and ```explorer .``` hint from the [Basic terminal interaction](#basic-terminal-interaction) section.)*

Insert the email address associated with your GPG key into the following command and run it in your terminal:
```
gpg --output gpg_key_public.txt --armor --export YOUR_GPG_EMAIL_ADDRESS
```

If you open the generated file in a text editor, you'll see that it starts with ```-----BEGIN PGP PUBLIC KEY BLOCK-----``` and it closes with ```-----END PGP PUBLIC KEY BLOCK-----```. When copying this key text to somewhere else later, like your GitHub account, you'll need to make sure that you **include these two lines** and everything between them.

Now let's get your **secret** GPG key into a file named ```gpg_key_secret.txt```. *(Again, this is the **secret** key only **you** should have access to, **DO NOT EVER** share this file with anyone else. I will only explain how to export this so that you can back it up in a safe place, and install it on your other computers.)*

Insert the email address associated with your GPG key into the following command and run it in your terminal:
```
gpg --output gpg_key_secret.txt --armor --export-secret-key YOUR_GPG_EMAIL_ADDRESS
```

If everything went well, you should now have three files:
- ```gpg_key_id.txt``` with your GPG key ID
- ```gpg_key_public.txt``` with your **public** GPG key that you can share with others, e.g. GitHub
- ```gpg_key_secret.txt``` with your **secret** GPG key that you **don't give to anyone else**

You may now proceed [to the next section](#add-public-gpg-key-to-github).

<br>

### The macOS GUI way

When you [installed the GPG tools](#installing-gpg-tools), an application titled "GPG Keychain" was added to your system. GPG Keychain is a visual key manager for macOS, it allows you to (among other things) create, export and import GPG key pairs. Run GPG Keychain now, the easiest way is probably to press *Cmd-Space* to open Spotlight and start typing "GPG Keychain" until it shows up in the result list. Alternatively, you can also use Finder to locate it in *Applications/GPG Keychain.app*.

With Kleopatra open, press the key combination *Cmd-N* on your keyboard (or select ```File > New Key...``` from the menu bar) to start the process of creating a new GPG key pair.

A dialog will open and ask you for your name, email address and a password.

As mentioned before, you don't **have** to provide your real name - unless there is a reason to, e.g. when using the GPG key pair for official email communication etc.

The email address should definitely be the one used to sign commits with later. Remember that you may have to use the weird GitHub no-reply email address here, rather than your actual main GitHub email address, depending on your [GitHub account email settings](https://github.com/settings/emails).

If you want to use a password to protect your **secret** key, then type it into the **Password** textbox. (Also in the second textbox that will show up as soon as you start typing.) *Reminder: you will need to enter this password every time your GPG key is used to sign a commit.* If you opt to not set a password, you will need to confirm your choice, just select ```Continue without password``` from the confirmation dialog.

Click the little chevron arrow icon next to **Advanced options** to fold out an area with further configuration options.

The **Comment** textbox is optional, you could use it e.g. to describe what this key pair was generated for.

The **Key type** dropdown is probably already set to ```RSA and RSA (default)```. If it's not, click the dropdown box and select this entry from the menu.

In the **Length** textbox underneath, specify the length of the key, which is ```4096``` by default.

If you don't wish to set an expiration date for your key pair, un-check the "Key will expire on" checkbox, else enter a date on which your GPG key pair will expire.

Close the dialog by clicking the **Create Key** button to create your new GPG key pair.

If the application asks you whether or not to upload your public key to an online key server, you will have to decide that for yourself. Purely for signing GitHub commits, it's not necessary to do this. If you want to communicate with others via GPG encrypted emails, you should consider doing this.

At this point, you should find yourself back in GPG Keychain's main window, your newly created GPG key pair should be displayed in the table. Don't close GPG Keychain yet, you'll need it again in a moment.

For now, let's figure out the exact ID of your GPG key, you might need this ID later to perform further actions on your key pair. Run the following command in your terminal: ```gpg --list-secret-keys --keyid-format=long```

This will print a list of all GPG keys currently registered on your system. If you see more than one entry listed, locate the one with the name and email address matching the ones you specified in the previous steps.

The first line of the entry should be in a format like ```sec   rsa4096/[RANDOM_LETTERS_AND_NUMBERS] [DATE] [LETTERS]```. The GPG key ID we're after is the *RANDOM_LETTERS_AND_NUMBERS* part. Double-click that block of letters and numbers to highlight/select it in your terminal, then press *CMD-C* (macOS) to copy them to your system clipboard. Paste the key into a text editor, and (for now) save the file somewhere as ```gpg_key_id.txt```.

The final step for this section will be to export your **public** and **secret** GPG keys into files. As mentioned before, I will use the file extension ```.txt``` here for ease of use, but you're welcome to use the standard ```.asc``` file extension for armored files instead. The actual file names don't matter, but I'll go with ```gpg_key_public``` and ```gpg_key_secret``` here to keep it simple and obvious.

First, let's get your **public** GPG key into a file named ```gpg_key_public.txt```. Go back to GPG Keychain and click the entry of your new key pair in the list to select it, then press the key combination *Cmd-E* on your keyboard (or right-click the entry and select ```Export...``` from the context menu).

A "Save As" dialog box will open. Navigate this dialog to the folder that contains the ```gpg_key_id.txt``` file you just created, then enter the full filename ```gpg_key_public.txt``` into the textbox at the top, make sure the "Include secret key in exported file" checkbox at the bottom of the dialog window is **NOT** checked, and finally confirm the export by clicking the **Save** button.

Now you'll repeat the above steps once more. Highlight the new entry in the list, press *Cmd-E*, navigate the dialog to the folder on your hard drive with the previously created ID and **public** key files. But this time, you'll enter the filename ```gpg_key_secret.txt``` instead, and you'll make sure the "Export secret key in exported file" checkbox at the bottom of the dialog **is CHECKED**. Then click the **Save** button to proceed with the export.

If everything went well, you should now have three files:
- ```gpg_key_id.txt``` with your GPG key ID
- ```gpg_key_public.txt``` with your **public** GPG key that you can share with others, e.g. GitHub
- ```gpg_key_secret.txt``` with your **secret** GPG key that you **don't give to anyone else**

You may now proceed [to the next section](#add-public-gpg-key-to-github).

<br>

### The Windows GUI way

When you [installed the GPG tools](#installing-gpg-tools), an application titled "Kleopatra" was added to your system. Kleopatra is a visual key manager for Windows, it allows you to (among other things) create, export and import GPG key pairs. Run Kleopatra now, the easiest way is probably to click the Windows Start button and start typing "Kleopatra" until it shows up in the result list.

With Kleopatra open, press the key combination *Ctrl-N* on your keyboard (or select ```File > New OpenPGP Key Pair``` from the application menu) to start the process of creating a new GPG key pair.

A dialog will open and ask you for your name and email address.

As mentioned before, you don't **have** to provide your real name - unless there is a reason to, e.g. when using the GPG key pair for official email communication etc.

The email address should definitely be the one used to sign commits with later. Remember that you may have to use the weird GitHub no-reply email address here, rather than your actual main GitHub email address, depending on your [GitHub account email settings](https://github.com/settings/emails).

If you want to use a password to protect your **secret** key (this will be the password you have to enter on every commit), then check the box at "Protect the generated key with a passphrase". (You'll be asked to provide the password later.)

Click the "Advanced settings..." button to bring up a dialog window with more configuration options.

In this dialog, pick the encryption methods ("Key Material") to use (*RSA 4096 bits + RSA 4096 bits are standard*) and what the **secret** key can be used for under "Certificate Usage". Make sure the option **Signing** is checked there.

If you don't wish to set an expiration date for your key pair, un-check the "Valid until" checkbox, else enter a date on which your GPG key pair will expire.

Close the "Advanced settings" dialog by clicking the **OK** button, then proceed with the GPG key pair creation by confirming the "Create OpenPGP Certificate" dialog box by clicking the **OK** button.

At this point, yet another dialog box should open, and finally ask you to enter the "passphrase to protect your new key". *Reminder: you will need to enter this password every time your GPG key is used to sign a commit.* If you opt to not set a password, you will need to confirm your choice, and you might have to repeat the "enter passphrase" and confirmation step once or twice. Just keep your input consistent.

Eventually, you should be presented with a success dialog that tells you "A new OpenPGP certificate was created successfully". *(You can ignore the fingerprint number for now, no need to take note.)*

Close the success dialog by clicking its **OK** button, and you should find yourself back in Kleopatra's main window, your newly created GPG key pair should be displayed in the table.

First, let's get your GPG key ID stored away. To do that, highlight the entry of your new GPG keys in the list, then left-click the cell with the numbers and letters at the very right side of the table, in the column "Key ID". There should now be a very thin dotted little frame around that cell. Now press *Ctrl-C* on your keyboard to copy them to your system clipboard. Paste the key into a text editor, remove the spaces between the segments, and (for now) save the file somewhere as ```gpg_key_id.txt```.

The final step for this section will be to export your **public** and **secret** GPG keys into files. As mentioned before, I will use the file extension ```.txt``` here for ease of use, but you're welcome to use the standard ```.asc``` file extension for armored files instead. The actual file names don't matter, but I'll go with ```gpg_key_public``` and ```gpg_key_secret``` here to keep it simple and obvious.

First, let's get your **public** GPG key into a file named ```gpg_key_public.txt```. Click the entry of your new key pair in the list, then press the key combination *Ctrl-E* on your keyboard (or right-click the entry and select ```Export``` from the context menu).

This will bring up an "Export OpenPGP Certificate" dialog, and you might notice that it only allows you to select one specific file type with the common OpenPGP file extensions. Don't worry about this, if you enter a filename with a specific file extension, e.g. ```.txt```, the dialog will respect it.

So navigate the dialog to the folder that already has your key ID text file in it, specify the filename ```gpg_key_public.txt``` at the bottom of the dialog, then press *Enter* or click the **Save** button to finally export the key.

Now let's get your **secret** GPG key into a file named ```gpg_key_secret.txt```. *(Again, this is the **secret** key only **you** should have access to, **DO NOT EVER** share this file with anyone else. I will only explain how to export this so that you can back it up in a safe place, and install it on your other computers.)*

Click the entry of your new key pair in the list, then right-click the entry and select ```Backup Secret Keys``` from the context menu. This will bring up a "Secret Key Backup" dialog that works the same way as the "Export OpenPGP Certificate" dialog a few lines up.

Just navigate the dialog to the folder with your ```gpg_key_id.txt``` and ```gpg_key_public.txt``` files already in it, then specify the filename ```gpg_key_secret.txt``` at the bottom of the dialog and press *Enter* or click the **Save** button to finally export the secret key.

You can now exit and quit the Kleopatra application.

If everything went well, you should now have three files:
- ```gpg_key_id.txt``` with your GPG key ID
- ```gpg_key_public.txt``` with your **public** GPG key that you can share with others, e.g. GitHub
- ```gpg_key_secret.txt``` with your **secret** GPG key that you **don't give to anyone else**

And with that out of the way, it's now time to add your **public** GPG key to your GitHub account.

<br>

## Add public GPG key to GitHub

So now that you have a **secret** key that you can sign your commits with, GitHub will need a way to verify that the commits they receive from someone claiming to be you were actually signed with the correct **secret** key. To be able to do that, GitHub will need to know about your **public** key.

### Adding the actual GPG key

Go to the [Keys page in your account settings](https://github.com/settings/keys) and locate the section titled **GPG Keys**. To the right of the **GPG Keys** heading, you should find a green button labelled **New GPG key**. Click this button.

You'll be taken to a very simple form, it has a "Title" textbox, a "Key" textarea, and a green "Add GPG key" button underneath.

In the "Title" textbox, enter a name or a comment that will help you identify this GPG key later. In most cases, the name or email address associated with your GPG key will be sufficient.

Locate the ```gpg_key_public.txt``` file you created earlier and open it in a text editor. Highlight the entire content of the file (*Cmd-A* on macOS or *Ctrl-A* on Windows) and then copy it to your system clipboard with *Cmd-C* (macOS) or *Ctrl-C* (Windows). Switch back to your browser and the GitHub form, then paste the file content into the "Key" textarea by first clicking into the textarea, and then pressing *Cmd-V* (macOS) or *Ctrl-V* (Windows) on your keyboard. Alternatively, you could also right-click into the "Key" textarea and then pick ```Paste``` from the context menu.

The "Title" textbox should say something helpful about what this key is, the "Key" textarea should contain your **public** GPG key in text form, including the surrounding lines ```-----BEGIN PGP PUBLIC KEY BLOCK-----``` and ```-----END PGP PUBLIC KEY BLOCK-----```.

Finally click the green **Add GPG key** button to submit your key to GitHub.

If everything worked and your key was accepted, the **GPG Keys** section should now list a new block with a key icon, the email address and name associated with your GPG key, the GPG key ID and the date on which it was added.

That's it. GitHub now knows how to verify the commits you'll push to it later.

<br>

### Vigilant Mode

Under the ***GPG Keys*** section on your account's [Keys page](https://github.com/settings/keys), you might notice another section named ***Vigilant Mode***, containing a single checkbox labelled *Flag unsigned commits as unverified*.

[Vigilant Mode](https://docs.github.com/en/authentication/managing-commit-signature-verification/displaying-verification-statuses-for-all-of-your-commits#about-vigilant-mode) does what its description says: if this checkbox is enabled, then just like any signed commits you make from now on will be visually marked as "verified" with a green badge, any commits to your repositories that are **not** signed will be visually marked with a yellow-brown-ish "unverified" badge.

If you're exclusively going to work with signed commits from now on, it can be beneficial to check the checkbox and enable Vigilant Mode from now on.

By default, (I think) this checkbox will be unchecked. With it unchecked, any unsigned commits will just be listed without any badge at all, neither "verified" nor "unverified".

To my understanding, this is purely a visual cue but does not actually stop anyone (who can) from pushing unsigned commits to your repositories. To stop that from happening, you would have to employ [branch protection or rulesets](#branch-protection-and-rulesets).

<br>

### Branch Protection and Rulesets

*This is an optional step, you don't have to do this. I am only going to elaborate on these features since they are topically related, build on the presence of GPG keys and signed commits, and they can help to improve the security within your repositories. If you don't care about these things, you're welcome to skip ahead [to the next section](#make-local-git-sign-commits-with-your-secret-gpg-key).*

If you're exclusively going to use GPG signed commits from now on, it might be a good idea to configure your repositories in such a way that they don't even accept unsigned commits to be pushed to them. This means if someone tries to push an unsigned commit, that push will be rejected and they'll get an error message.

You can enforce the rejection of unsigned commits on a per-branch basis in each repository individually, and you do so by setting up either Branch Protection rules or Rulesets.

Branch Protection rules and Rulesets are (the clue is in the name) sets of rules that can be enforced inside a repository to protect code branches. They will need to be configured for each repository individually, as each repository can have different requirements. They both do similar things but in different ways and to different extent. Somewhat simplified, Branch Protection rules are static and unique, but Rulesets can be combined and layered.

Which way you choose to go is up to you, I guess it depends on the specific requirements of your repositories, or the complexity of the rules, or your personal preferences. For the purpose of just rejecting unsigned commits, either method should be fine. If you e.g. already have a compatible Branch Protection rule active, you may want to extend that rather than introduce further complexity by adding Rulesets.

*For the scope of this instructional, I will assume that your repository has neither feature configured yet, and I will focus on creating a **Ruleset**.*

Go to your repository, then go to its **Settings** page. In the sidebar on the left, you will see an entry ```Rules```. Click it, then click on the ```Rulesets``` option that unfolds.

On the **Rulesets** page, click the green **New Ruleset** button in the upper right corner. From the dropdown menu, decide if your want to set up a Ruleset to protect branches or tags. In this case, go with ```New branch ruleset```.

Give the Ruleset a fitting name, maybe something like "Signed commits only".

Click the **Enforcement status** dropdown and select ```Active``` from the list, this means the rule will be enforced right away after setting it up.

In the **Target branches** section, click the ```+ Add target``` button and pick an option from the dropdown menu. For the sake of simplicity, let's go with ```Include all branches``` so that the entire repository will be protected.

In the list of **Branch protection** features, enable the ones you want, disable the ones you don't want, it's up to you. To make your repository reject unsigned commits, the only checkbox that really needs to be checked is the one that says ```Require signed commits```, all the others could theoretically be unchecked.

Finally, confirm your configuration by clicking the green **Create** button at the bottom of the page. This will create the new Ruleset and, depending on the status of your account and/or visibility of your repository, start enforcing it right away.

From now on, any unsigned commits made to your repository will be rejected.

**NOTE** that Branch Protection and Rulesets are features that are limited to **public** repositories or paid GitHub Team/Enterprise accounts. If you're using GitHub with a free account, you can define Branch Protection rules and Rulesets for your **private** repositories if you wish, but the rules will not be enforced until the repository's visibility is switched to **public**. Once a repository owned by a Free account is set to **public**, any configured Branch Protection rules and Rulesets will start working.

<br>

## Make local Git sign commits with your secret GPG key

Now you have a system-wide Git installation, a system-wide GPG installation, you've created a GPG key pair, you've added the **public** GPG key to your GitHub account, and maybe you've increased the security level of your repositories by enabling [Vigilant Mode](#vigilant-mode) or a [Ruleset](#branch-protection-and-rulesets).

The last missing piece now is to configure your local Git installation so that it actually attempts to sign your commits.

### Tell Git where GPG is located

Let's start by telling your Git installation where it can find the GPG executable file. It probably makes sense to add this setting **globally**, so that your Git installation generally knows where to find the GPG program if one of your repositories requires it - even if you don't enable signing for all your repositories. (More on that later.)

In the [Installing GPG tools](#installing-gpg-tools) section, I asked you to make a note of where the GPG tools installer actually puts the files, at least on **Windows**. Right now, this is where you'll need that information.

On Windows, the default installation path of the required file would be this:
```
C:\Program Files (x86)\GnuPG\bin\gpg.exe
```

On macOS, the installation path of the required file would be this:
```
/usr/local/bin/gpg
```

To inform your Git installation of which executable file to use to sign commits, open a terminal window and run the following command, but substitute the placeholder with the actual path to the GPG executable on your system:
```
git config --global gpg.program "[PATH_TO_GPG_EXECUTABLE]"
```

### A note about Global or Individual settings

There are two ways you can set your repositories up for signing commits, either **per individual repository** or **globally for all Git repositories**.

If you want ALL your commits to ALL your Git repositories to get signed, then go for the **global** method.

However, if some of your repositories use different identities, you might **not** want to set this up fully globally, but possibly using individual GPG keys (or none) for each repository.

In the terminal commands below, I will always mention ```[--global]``` as an indicator that you **can** insert the ```--global``` flag at this position to make the settings apply to **all** your repositories, but you don't have to. If you leave the global flag out, then the setting will only apply to the repository located at **your terminal's current working directory**.

### Configuring a Git repository to sign commits

Now that your Git installation knows where to find the GPG executable, which enables it to sign things and access the private GPG keys known to your system, it's time to set at least one of your checked-out repositories up to sign your commits. You'll need to instruct Git to sign commits, to sign tags (if you wish), and you'll need to instruct it which key to actually use when signing.

Open a terminal window, navigate to a Git repository in which you want to switch signed commits on, then run this command:
```git config [--global] commit.gpgsign true```

If you also wish to sign new tags, run this command as well:
```git config [--global] tag.gpgsign true```

To let Git know which GPG key it should use to sign your commits, remember the GPG key ID you exported earlier into ```gpg_key_id.txt```. Insert it into the following command and run it:
```git config [--global] user.signingkey [YOUR_GPG_KEY_ID]```

And that's it.

From now on, any Git client that uses the system-wide Git version should automatically sign your commits.
All you need to do now is start committing.

Easy, right? :)

<br>

## Concluding Summary

1) You installed a system-wide Git version on your machine.
2) You installed a system-wide GPG version on your machine.
3) You created GPG keys and exported them to files that allow you to install them on other machines as well.
4) You added your **public** GPG key to your GitHub profile, and possibly activated some new related security features.
5) You configured your Git installation, and maybe some repositories, to sign commits using your **private** GPG key from now on.

Although it was a tough and long read, because I tend to blather on at times, I hope this instructional not only helped you to get your development machine to send signed commits to GitHub, but that it also helped you understand what you were (and will be) doing.

If you have any comments or suggestions, please report and discuss them in the [Issues section](https://github.com/rcliftonharvey/gpg-for-github/issues) of this repository.

<br>

## Further resources

Just a set of links that helped me understand what's going on, included here for future reference.

* https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work
* https://docs.github.com/en/authentication/managing-commit-signature-verification
* https://gist.github.com/xavierfoucrier/c156027fcc6ae23bcee1204199f177da#file-gpg-signing-md
* https://stackoverflow.com/questions/42675983/commit-signing-using-sourcetree-on-windows
* https://stackoverflow.com/questions/36810467/git-commit-signing-failed-secret-key-not-available
* https://confluence.atlassian.com/sourcetreekb/setup-gpg-to-sign-commits-within-sourcetree-765397791.html
* https://www.youtube.com/watch?v=u9L5kDlU8rs
