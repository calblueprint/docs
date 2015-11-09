Developer Workflow
====
Phabricator best practice is to do all your work in branches. You do no development in the master branch; that is just a staging branch to push upstream (to GitHub).
Some of these rules are stricly enforced by the above configuration you did on your local repo.

1. When you want to start work on a new feature or change, create a local branch:

		$ git checkout -b dev-name/cool-new-feature

	It's not necessary to prepend the branch name with your name, but it might make it easier for your PL to keep track of who is working on what when testing
	out multiple changes.

2. Make your sexy changes, add all the files, and commit normally (don't use the `-m` flag):

		$ git commit

3. Now you should be ready for a review. Run:

		$ arc vdiff

	The automated script you just ran should have set up this arc alias for you. We use `arc vdiff` instead of `arc diff` because we want to explicitly keep the format of our commit template. (Run `arc diff --help` for more detail on this, or ask me @aleks).

	When commiting for the first time, the template will prompt you for a quick description, a more lengthy summary, and a test plan. Traditionally, you should put a
	very **brief** description in the first line and then expand upon what you wrote in the summary. The idea here is that somebody that quickly reads the summary line
	should be able to get an idea of what your diff is about. Lastly, the test plan should include some brief instructions on how your PL can verify that your code
	works (e.g. `rake db:migrate; rake db:seed`)

	- **Super useful:** If you'd like to preview your diff before actually submitting it for review, use `arc diff --preview` instead! This will pop up a phabriactor URL with
	a pretty code review UI without notifying your PL or team.

4. Often, your PL will give you feedback on your review that will prompt you to make changes before you can push to master.  Make the changes/fixes as normal in your branch, and then use `git commit --amend` to amend your previous git commit. Then, run `arc vdiff`. Your editor should open up with a message indicating that your are making a revision to an _existing_ Phab diff.
	>
	- If you don't need to change your commit message on amend, you can use  `git commit --amend --no-edit` to skip editing it.
	- If you need to force `arc` to assign the new commits to the correct Phab revision ID, use `arc diff --update <revision id>`

5. When you are ready to push, you'll want to rebase your changes into the master branch and do a git push from master up to GitHub. (See the note below on rebasing.) You can do this in one step, which will also update your commit message with Phabricator metadata: `arc land`
	- If you want to be explicit, you can run `arc land <to-branch>`

	> **Rebasing:** Our Phabricator projects are set up to `rebase` instead of `merge` by default; this should have been automatically configured. Rebasing gives the illusion of a much cleaner commit history. Ask your PL to clarify the difference between `merge` and `rebase`, you should become somewhat familiar with the two.

Often you'll want to incorporate new changes into your branch that people have committed to master after you created your branch. In this case you'll want to pull the newest changes from master (`git checkout master; git pull`) and then rebase your branch on top of the newest changes (`git checkout <feature-branch>; git rebase master`).

See the Phabricator authors' workflow for more in-depth, detailed info [here](https://secure.phabricator.com/w/guides/arcanist_workflows/).

### Summary
	$ git checkout -b <feature branch>
	<edit edit edit>
	$ git commit
	$ arc vdiff

	<edit based on review feedback>
	$ git commit --amend
	$ arc vdiff

	<maybe run git pull if you want some updates>
	<edit + commit + arc diff some more>

	$ arc land

### Frequently Asked Questions (FAQ)

1. **Why do I even want to use Phabricator? Github pull requests and issues provides all the functionality I need.**

	Though Phabricator doesn’t offer any essential new functionality, we encourage switching over for 4 main reasons:

	- It’s difficult to keep track of Github comments since they tend to disappear when you make a new revision. Alternatively, Phabricator has a really sleek UI for viewing
	old comments in context.
	- Phabricator lets you see the code differences from incremental revisions. It’s sometimes convenient to see what changes a specific update introduced, especially in
	large revisions, so phabricator makes it really easy to diff against arbitrary versions.
	- Phabricator’s code review UI is quite sexy
	- Phabricator supports dank memes

2. **Why am I being prevented from committing to master?**

	We want **all** of our changes to go through some sort of code review. Even if seems like a ridiculously trivial change, it's often better to get a better set of eyes on
	your code instead of commiting directly to master. We included the pre-commit check since even if you're always trying to work on feature branches, it's possible to
	momentarily forget what branch you're on.

3. **Why can't I commit with my own custom message?**

	It's always nice to be able to get a general sense of what changes a specific commit introduced when looking at the message. This property is extremely useful in large
	codebases for debugging and general traversal purposes. Our suggested workflows include a detailed commit template and ameding commits which should make it a little
	easier. Though it might surprise you, an outsider won't learn a lot from reading a commit message like `oh shit it works!!1!`

4. **I edited an existing diff, but arcanist's template assumes I'm creating a new diff. How do I fix this?**

	Sometimes arcanist gets a little bit confused (seems to happen more often with amending commits), but luckily there's a very easy solution. Simply find you differential
	number (should be a capital 'D' followed by a number) and run `arc diff --update D{rev_number}`. This should bring up the correct template that updates a diff.

5. **Why do I want to amend my commits instead of creating a new one?**

	The rationale here is similar to question 3 above, but `amend` ensures that all of the changes associated with a diff are encapsulated in single commit.
	Additionally, amending commits traditionally leads to a cleaner commit history. If you accidentally made additional commits but want your final push to only have
	one commit, you can look into [squashing commits](https://ariejan.net/2011/07/05/git-squash-your-latests-commits-into-one/).

