Developer Workflow
====
Phabricator best practice is to do all your work in branches. You do no development in the master branch; that is just a staging branch to push upstream (to GitHub).
Some of these rules are stricly enforced by the above configuration you did on your local repo.

1. When you want to start work on a new feature or change, create a local branch:

		$ git checkout -b cool-new-feature

2. Make a bunch of changes, and commit normally (don't use the `-m` flag):

		$ git commit

3. Now you should be ready for a review. Run:

		$ arc vdiff

	The automated script you just ran should have set up this arc alias for you. We use `arc vdiff` instead of `arc diff` because we want to explicitly keep the format of our commit template. (Run `arc diff --help` for more detail on this, or ask me @aleks).
	- **Super useful:** If you'd like to preview your diff before actually submitting it for review, use `arc diff --preview` instead!

4. Often, your PL will give you feedback on your review that will prompt you to make changes before you can push to master.  Make the changes/fixes as normal in your branch, and then use `git commit --amend` to amend your previous git commit. (This is normally bad GitHub practice, but becomes very useful and actually preferred when using Phabricator.)  Then, run `arc vdiff`. Your editor should open up with a message indicating that your are making a revision to an _existing_ Phab diff.
	>
	- If you don't need to change your commit message on amend, you can use the `--no-edit` flag to skip editing it.
	- If you need to force `arc` to assign the new commits to the correct Phab revision ID, use `arc diff --update <revision id>`

5. When you are ready to push, you'll want to rebase your changes into the master branch and do a git push from master up to GitHub. (See the note below on rebasing.) You can do this in one step, which will also update your commit message with Phabricator metadata: `arc land`
	- If you want to be explicit, you can run `arc land <to-branch>`

	> **Rebasing:** Our Phabricator projects are set up to `rebase` instead of `merge` by default; this should have been automatically configured. Rebasing gives the illusion of a much cleaner commit history. Ask your PL to clarify the difference between `merge` and `rebase`, you should become somewhat familiar with the two.

If other people have committed changes that you want to incorporate into your branch (perhaps a bugfix that you need), you can just do `git pull --rebase` in your feature branch (or simply `git pull` since your repo has set up rebase by default).  Phabricator will do the right thing in ignoring these not-your changes.

See the Phabricator authors' workflow for more in-depth, detailed info [here](https://secure.phabricator.com/w/guides/arcanist_workflows/).


### Summary
	$ git checkout -b <feature branch>
	<edit edit edit>
	$ git commit
	$ arc vdiff

	<edit based on review feedback>
	$ git commit
	$ arc vdiff

	<maybe run git pull if you want some updates>
	<edit + commit + arc diff some more>

	$ arc land

