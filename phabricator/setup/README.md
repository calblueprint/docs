Phabricator Setup
====
Phabricator is a platform for code review, much like GitHub Pull Requests. Phabricator, however, has a much more extensive and intuitive feature set. We'll be replacing Pull Requests with Phabricator Diffs!

###Table of Contents
1. [Developer Setup](#dev-setup)
2. [PL/Project Setup](#pl-setup)
3. [Appendix](#appendix)

External: [Developer Workflow](http://git.io/phab-at-bp)

<a name="dev-setup"></a>Developer Setup
----
### Step 1: Install Arcanist
Phabricator is the web tool for _doing_ code reviews, but Arcanist (or `arc`) is the command line tool that interacts with Phab. To install `arc`, you'll need to clone the GitHub repo and add the program to your `$PATH`. We're going to accomplish this by symlinking the program to your path. (**IMPORTANT:** Please read [section 4](#appendix-4) in the appendix if you don't know what this means!!! Or ask your PL.)

**IMPORTANT:** We'll be installing a few tools throughout this guide, so you'll need to pick/create a permanent directory to keep all of these tools. This guide will assume you'll be using the NIX-compilent directory `/usr/local/opt`. If you'd like to place the tools in a different directory instead (e.g. `~/.webdev`) be sure to follow the guide carefully and replace the default directory with your custom one where needed.

Once you've chosen your preferred directory, `cd` to it:

	$ cd /usr/local/opt

Next, let's clone and configure some of our tools by running the following:

    $ git clone https://github.com/calblueprint/libphutil.git
    $ git clone https://github.com/calblueprint/arcanist.git
    $ ln -s $(pwd)/arcanist/bin/arc /usr/local/bin/arc

(The last command above assumes that `/usr/local/bin` is already in your `PATH`; it should be.)

**Restart your terminal.** Now, when you run `arc`, you should see something like:

    Usage Exception: No command provided. Try 'arc help'.

If you instead see something like: `bash: command not found: arc`, something went wrong. Run through the steps again, or ask your PL for help.

> NOTE: We use a fork of both arcanist and libphutil. Make sure you clone our custom
forks, not the phalicity repos.

### Step 2: Clone and configure Traphic
We'll be using [Traphic](https://github.com/calblueprint/traphic.git) to enable Github-ready Continuous Integration for our projects. (**IMPORTANT:** Please read [section 5](#appendix-5) in the appendix if you don't know what continuous integration is, or ask your PL!) Traphic is a small external library for Arcanist that automagically pushes your feature branches to GitHub whenever you create a diff for Phabricator. (More on this later.)

First, make sure you're still in `/usr/local/opt`. Now, clone the Traphic repo:

    $ git clone https://github.com/calblueprint/traphic.git

Next, you'll need to export the environment variable `TRAPHIC_PATH` in your shell environment. Let's find out what kind of shell you're using to set this up correctly. Copy/paste/run the following in your terminal:

	$ echo $SHELL

Now, follow the directions that correspond to the output from the above command you just ran:

- Output: `/bin/bash`
	- You are running a `bash` shell! Open `~/.bash_profile` in your preferred editor and continue to the step after these bullets.
- Output: `/bin/zsh`
	- You are running a `zsh` shell! Open `~/.zshrc` in your preferred editor and continue to the step after these bullets.
- Output: something else
	- You probably know what you're doing. Open up your shell's environment setup file and continue.

At this point, you should have opened a configuration file for your shell in some editor. At the bottom of the file, paste the following line:

	export TRAPHIC_PATH=/usr/local/opt/traphic

**IMPORTANT:** Relative paths (i.e. `~/.webdev`) are not allowed here!

You've just exported the `TRAPHIC_PATH` environment variable into your shell environment. **Restart your shell before continuing.**
> **SUPER IMPORTANT:** If you don't understand what "exporting an environment variable" actually means, read [section 2](#appendix-2) in the appendix or ask your PL to clarify!

Assuming your PL set up your project correctly, you should be good to go with Traphic.


### Step 3: Have your PL make you a Phabricator account
**IMPORTANT:** Make sure that you've already authenticated an `@berkeley.edu` account with your GitHub before proceeding!

Ask your PL to add you as a user on our Phabricator. _Ask your PL to set your Phabricator username to your Slack username, not your GitHub username!!!_

You should recieve an email with an authentication link. Follow that link, and you should be good to go.

### Step 4: Install Phabricator certificate
Navigate to the root of your project's directory, e.g.

	$ cd /path/to/project_homeless_connect

At this point, I'm assuming that your PL has already configured your project correctly. Just in case, checkout to master, `git pull` the latest changes, and run `cat .arcconfig`. **This should output a JSON configuration file.** If you see a blank output or an error, _then your PL may not have configured your project for Phabricator yet_! Please bring this up with them or me (@aleks).

Assuming everything is correctly configured, from the root of your repository, run the following:

	$ arc install-certificate

This should print a URL to your terminal. Follow the link, copy the code given, and paste it back in your terminal.

### Step 5: Configure local repo git conventions
Luckily, this step is automated. In the root of your repo and copy/paste the following into your terminal to curl and run an automated script that will set up your repo with some goodies that should streamline your workflow:

	$ python <(curl -sL http://git.io/vWg1q)

This script does the following automagically:

- Configures a git hook on `pre-commit` which disables committing to master, and also promps you if you're trying to make a new commit on top of an existing Phab diff.
- Configures a git hook on `pre-push` which disables pushing to master.
- Sets up a custom commit message template which defaults to adding your PL and teammates as Reviewer and CCs, respectively.
- Configures a git hook on `prepare-commit-msg` to disable committing with `-m`, in order for git to pick up the aforementioned commit template.
- Sets up your repo to automatically `rebase` on `git pull`.
- Configures `arc vdiff`, an alias for `arc diff --verbatim`.

Don't worry, all of these configuration will be local to _this_ git repo (and only on _your_ computer). None of your other git repos will be affected!

> **A NOTE FOR PLs:** These restrictions can be bypassed if there is an emergency! Please ping me (@aleks) on Slack for details if you forget how to.


### Step 6: Voila!
You should be configured to push diffs for code review on Phabricator, yay! Please check out the [Developer Workflow guide](http://git.io/phab-at-bp) for proper Phabricator practice.

<a name="pl-setup"></a>(PLs) Project Setup
----
Read this section if you're a PL setting up your project with Phabricator.

### Step 1: Make an admin Phabricator account
> **NOTE:** Make sure that you've already authenticated an `@berkeley.edu` account with your GitHub before proceeding!

Go to [phab.calblueprint.org](http://phab.calblueprint.org) and authenticate with GitHub. Ask me (@aleks) or someone else that already has admin access to activate your account and give you admin privilege. You'll need admin privilege in order to activate your devs later.

### Step 2: Add an `.arcconfig` file to your project's repo.
(This part assumes you already have a GitHub repo.)
Make a new `.arcconfig` file in the root of your repo that points to our Phab.

	{
	  "phabricator.uri": "http://phab.calblueprint.org",
	  "history.immutable": false,
	  "load": [
	    "$TRAPHIC_PATH/traphic"
	  ],
	  "arcanist_configuration": "TraphicConfiguration"
	}

Later on, you'll probably want to add linters and unit test engines to your configuration files. This lets you run lint and unit tests before making a
Phab diff (which is awesome, because we can enforce good style and not deal with Hound). More information on this
[here](https://secure.phabricator.com/book/phabricator/article/arcanist_new_project/).

For most Rails + React projects, we are probably going to want to set up default linters for both our Ruby and Javascript code.
Luckily, arcanist comes packaged with the Rubocop linter, but it unfortunately doesn't have great support for JSX code.
Therefore, the rest of this guide assumes that you have our forked copy of arcanist, since we've included a custom ESLinter for you.

From here, setting up the linters is very easy. For example, if you wanted to have linters for Ruby, Javascript, and CSS, you would create an
`.arclint` file in the root of your project directory like:

	{
	  "linters": {
	    "ruby": {
	      "type": "rubocop",
	      "include": "/\\.(rb|rake)$/",
	      "exclude": "(^db/)",
	      "rubocop.config": ".rubocop.yml"
	    },
	    "jsx": {
	      "type": "eslint",
	      "include": "/\\.(js|jsx)$/"
	    },
	    "csslint": {
	      "type": "csslint",
	      "include" : "(\\.css$)"
	    }
	  }
	}

> **NOTE**: Due to the way the linters are configured, you will have to define which paths to check in this file and NOT in a different config file.
  Therefore, the include / exclude rules you have defined in a `rubocop.yml` will not apply. However, all other rules will work as expected.

### Step 3: Configure Diffusion for your project
Phabricator uses a tool called Diffusion to track your GitHub project commits, assigning useful metadata to your diffs.
GitHub will still host all of our projects, Phabricator simply watches the GitHub repo. Diffusion needs to be
configured for things to work properly with Phabricator.

- Go to [http://phab.calblueprint.org/project/](http://phab.calblueprint.org/project/) and click the "New Project" button in the upper-right corner of the page.
- Choose your project name (e.g. "California Rangeland Trust") and a super short tag for your commits (e.g. `CRT`).
- You've just created a tag for your project. You can also do cute things like change your tag's color or icon. **IMPORTANT:** Later, once your devs have been set up, you'll want to add them as members of your project.
- Go to [http://phab.calblueprint.org/diffusion/](http://phab.calblueprint.org/diffusion/) and click the "New Repository" button in the upper-right corner of the page.
- Choose to "Import an Existing External Repository" and follow the instructions until the "Repository Ready!" step. At this point, choose "Configure More Options First"

	> If you missed this step, find your way back to your project's Diffusion settings page by
	going to your project's page in Diffusion and clicking "Edit Repository" on the right side
	of the project info card.

- In the first box, click "Edit Basic Information" on the right.
	- Add your previously-created project tag to this repository, and save your changes.
- Scroll down to the "Branches" section, and click the "Edit Branches" button.
	- In the "Track Only" box, paste the following:

			regexp(/^(?!TR\_D)/)

	- In the "Autoclose Only" box, paste the following:

    		master

	- Save changes.
- Scroll back up to the top of the page and press "Activate Repository", in the first box on the right. (You may have already done this earlier.)

All done!

### Step 4: Configure local repo
- Follow the above [Developer Setup](#dev-setup) guide.
- **As a PL, you should become familiar with the points in the [appendix](#appendix).**
- Also as a PL, brush up on proper [Phabricator workflow](http://git.io/phab-at-bp).

### Step 5: Profit!
Prepare to enter code-review ~~hell~~ heaven!

**IMPORTANT:**

- Make sure your devs have git >= 2.0 installed!
- Make _absolute_ sure that all of your devs curl and run the automated set-up script we've provided. There could be serious confusion otherwise. This way, no dev can accidentally spoil your ~~beautiful~~ linear history.

<a name="appendix"></a>Appendix
----
1. **What's a symlink?**

	_TL;DR_: Symlinking creates a link to a file in another place. For example `ln -s /blah/bar.txt foo.txt` makes `foo.txt` link to `/blah/bar.txt`.

	"Symlink" is short for _symbolic link_, or _soft link_. Here's an explanation by example: Suppose that in your home directory `/Users/unzunz` you have an important file called `blooprint.txt`. But, for some reason, you only ever access that file when working on your side project in `/Users/unzunz/projects/awesome_app`. When you're working on `awesome_app`, you find it annoying that you always have to navigate back to your home directory to access `blooprint.txt`. To fix this issue, you can make a **symlink** to `blooprint.txt` from inside your app directory, like so:

		$ cd /Users/unzunz/projects/awesome_app
		$ ln -s /Users/unzunz/blooprint.txt pls.txt

	This creates a symlink `pls.txt` in `/Users/unzunz/projects/awesome_app` that _points_ to `/Users/unzunz/blooprint.txt`! You can view, edit, or run the file from either side.
	> If you delete `/Users/unzunz/blooprint.txt`, the `pls.txt` symlink will be broken; in other words, `pls.txt` will point to nowhere. This is why symlinks are also called "soft" links.

2. <a name="appendix-2"></a>**What is an environment variable? What does exporting one do?**

	Environment variables are variables present in your **shell environment**. By "shell" I mean your `bash`, `zsh`, etc. Every time you start a new shell instance, your shell will read through a set of configuration files (e.g. `~/.bash_profile`) and load a bunch of things into your shell environment. Things that are loaded include aliases, functions, and **variables**, among other things. A shell's environment variables are accessible from any program running in it.

	For example, `SHELL` is an environment variable that stores the name of the shell you are using. Running `echo $SHELL` will output its value.

	Adding `export FOO=blah` to one of your shell's configuration files will export/add the variable `FOO` with value `blah` to your shell environment! _However, you'll have to restart your shell in order for this to take effect._
	> Every time you restart your terminal, you load a new shell instance.

3. **What does `PATH` mean?**

	_TL;DR_: All the programs in your `PATH` can be run by just their name. For example, Homebrew is actually in `/usr/local/bin/brew`, but because it's in your `PATH`, you can just run it with `brew`.

	`PATH` is probably the most important **environment variable** for your shell. Read the point above if you don't know what environment variables are. `PATH` is a list of directories in the following format:

	`/usr/local/bin:/usr/bin:/some/other/directory:<more things>`

	Directories are delimited by `:`. `PATH` is a _special_ variable---when your shell loads the `PATH`, it "sources" all the executable programs in each directory given. This is why you can run `ls foo` or `cd foo` in your shell **without** explicitly running `/bin/ls foo` or `/bin/cd foo`.



4. <a name="appendix-4"></a>**What does adding a symlink to my `PATH` do?**

	If you're not familiar with symlinks and the `PATH`, read the three points above.

	Symlinking a program to your `PATH` effectively sources that program! For example, if I have a command line program at `/Users/Aleks/projects/bin/same`, instead of always having to explicitly run the following:

		$ /Users/Aleks/projects/bin/same arg1 arg2

	I can symlink `same` to somewhere in my `PATH` and run it in a much nicer way:

		$ ln -s /Users/Aleks/projects/bin/same /usr/local/bin
		$ same arg1 arg2

5. <a name="appendix-5"></a>**What is Continuous Integration and why is it important?**

	A Continuous Integration service (like Codeship) will watch a code repository and automatically run a suite of tests against it whenever a new change is made. Codeship and Travis (both CI services) hook directly into GitHub and runs tests on all branches whenever you push a change.

	CI is super important because it runs tests on every single iteration of your product, catching bugs immediately. Of course, you'll have to write tests first!
