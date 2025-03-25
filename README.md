# funny-little-rerere-example-repo
I'm investigating git rerere and I've found I want to replicate the idea presented in https://stackoverflow.com/a/77453543 by daiwai, which I feel is at odds with the details presented in https://stackoverflow.com/a/5519698 by VonC.

Anything I create for this project I place into the public domain. However, the code commits I'm going to make are from daiwai and since they're from stackoverflow they're CC BY-SA 4.0

I'm using git version 2.46.2.windows.1

# Conclusion and process

Conclusion: there is something funny going on. Also, daiwai was correct.

I rebased to update the "Update README.md: elaborate on project." commit (6c30b9d, initially), setting that up using git rebase --rebase-merges --root -i

 Here's the final state of the project after my experiment, after the rebase, from git log --all --graph --oneline:

```git
*   39ad442 (HEAD -> master, tag: post-rebase_structure) Merge branch 'another_branch' (this is the post-rebase merge)
|\
| * 927133f Conflicting Change
* | 30abd95 Refactoring
|/
* 06cada7 Initial state
* df03eb1 Update README.md: elaborate on project.
| *   51fb423 (tag: old_structure) Merge branch 'another_branch'
| |\
| | * d5e2c57 (another_branch) Conflicting Change
| * | 4b97fc9 Refactoring
| |/
| * 9ee603e Initial state
| * 6c30b9d (origin/master, origin/HEAD) Update README.md: elaborate on project.
|/
* 76bb797 Initial commit
```

So that's fine. However, I had to achieve this manually, basically. Here's a cleaned-up version of my git command-line interactions after finalizing the the change to the "Update README.md" commit that started the rebase:

```bash

$ git status # Nothing interesting has happened yet.

interactive rebase in progress; onto e386d81
Last commands done (4 commands done):
   pick 76bb797 Initial commit
   edit 6c30b9d Update README.md: elaborate on project.
  (see more in file .git/rebase-merge/done)
Next commands to do (7 remaining commands):
   pick 9ee603e Initial state
   label branch-point
  (use "git rebase --edit-todo" to view and edit)
You are currently editing a commit while rebasing branch 'master' on 'e386d81'.
  (use "git commit --amend" to amend the current commit)
  (use "git rebase --continue" once you are satisfied with your changes)

nothing to commit, working tree clean

$ git rebase --continue # OK, let's move on to the interesting commits...

Auto-merging a.js
CONFLICT (content): Merge conflict in a.js
Resolved 'a.js' using previous resolution.
Could not apply 51fb423... another-branch # Merge branch 'another_branch'

$ git status # A strange thing, seen below, is that a.js is marked as "both modified", but actually a.js is fine! No conflict markers in it at all. It's b.js that is wrong. B's missing the 'new line' line. It also doesn't have any conflict markers in it.

interactive rebase in progress; onto e386d81
Last commands done (11 commands done):
   pick 4b97fc9 Refactoring
   merge -C 51fb42311ffd0987fff11461ef66980955535ef3 another-branch # Merge branch 'another_branch'
  (see more in file .git/rebase-merge/done)
No commands remaining.

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   a.js

no changes added to commit (use "git add" and/or "git commit -a")

$ git checkout 51fb42311ffd0987fff11461ef66980955535ef3 -- . # After some thought, I decided to fix the merge conflict with a brute-force solution: just check out every file from the original post-merge commit.

$ git status # I check the status, and then realize that I've also checked out the README.md from the other commit, undoing the change that was the whole point of this rebase.

interactive rebase in progress; onto e386d81
Last commands done (11 commands done):
   pick 4b97fc9 Refactoring
   merge -C 51fb42311ffd0987fff11461ef66980955535ef3 another-branch # Merge branch 'another_branch'
  (see more in file .git/rebase-merge/done)
No commands remaining.

All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
        modified:   README.md
        modified:   b.js

$ git g # I use an alias for a command similar to git log --all --graph --oneline to see what all the commits are.

* 2025-03-25 01:59 (Tue) 927133f Conflicting Change <wyattscarpenter>
| * 2025-03-25 01:58 (Tue) 30abd95 (HEAD) Refactoring <wyattscarpenter>
|/
* 2025-03-25 01:53 (Tue) 06cada7 Initial state <wyattscarpenter>
* 2025-03-25 01:50 (Tue) df03eb1 Update README.md: elaborate on project. <wyattscarpenter>
| * 2025-03-25 02:07 (Tue) e386d81  <wyattscarpenter>
| *   2025-03-25 02:02 (Tue) 51fb423 (tag: old_structure, master) Merge branch 'another_branch' <wyattscarpenter>
| |\
| | * 2025-03-25 01:59 (Tue) d5e2c57 (another_branch) Conflicting Change <wyattscarpenter>
| * | 2025-03-25 01:58 (Tue) 4b97fc9 Refactoring <wyattscarpenter>
| |/
| * 2025-03-25 01:53 (Tue) 9ee603e Initial state <wyattscarpenter>
| * 2025-03-25 01:50 (Tue) 6c30b9d (origin/master, origin/HEAD) Update README.md: elaborate on project. <wyattscarpenter>
|/
* 2025-03-25 01:46 (Tue) 76bb797 Initial commit <wyattscarpenter>

$ git checkout df03eb1 -- README.md # Check out the version of README.md that I know is good: the one from the rebase commit.

$ git status

interactive rebase in progress; onto e386d81
Last commands done (11 commands done):
   pick 4b97fc9 Refactoring
   merge -C 51fb42311ffd0987fff11461ef66980955535ef3 another-branch # Merge branch 'another_branch'
  (see more in file .git/rebase-merge/done)
No commands remaining.

All conflicts fixed but you are still merging.
  (use "git commit" to conclude merge)

Changes to be committed:
        modified:   b.js

$ git commit # I also edit the commit message in the editor after doing this.

[detached HEAD 39ad442] Merge branch 'another_branch' (this is the post-rebase merge)

$ git status

interactive rebase in progress; onto e386d81
Last commands done (11 commands done):
   pick 4b97fc9 Refactoring
   merge -C 51fb42311ffd0987fff11461ef66980955535ef3 another-branch # Merge branch 'another_branch'
  (see more in file .git/rebase-merge/done)
No commands remaining.
You are currently editing a commit while rebasing branch 'master' on 'e386d81'.
  (use "git commit --amend" to amend the current commit)
  (use "git rebase --continue" once you are satisfied with your changes)

nothing to commit, working tree clean

$ git rebase --continue # This ends everything, 

Successfully rebased and updated refs/heads/master.

$ git g # This is essentially the final state of the repo, which I've already included at the top of this section.

*   2025-03-25 02:21 (Tue) 39ad442 (HEAD -> master) Merge branch 'another_branch' (this is the post-rebase merge) <wyattscarpenter>
|\
| * 2025-03-25 01:59 (Tue) 927133f Conflicting Change <wyattscarpenter>
* | 2025-03-25 01:58 (Tue) 30abd95 Refactoring <wyattscarpenter>
|/
* 2025-03-25 01:53 (Tue) 06cada7 Initial state <wyattscarpenter>
* 2025-03-25 01:50 (Tue) df03eb1 Update README.md: elaborate on project. <wyattscarpenter>
| *   2025-03-25 02:02 (Tue) 51fb423 (tag: old_structure) Merge branch 'another_branch' <wyattscarpenter>
| |\
| | * 2025-03-25 01:59 (Tue) d5e2c57 (another_branch) Conflicting Change <wyattscarpenter>
| * | 2025-03-25 01:58 (Tue) 4b97fc9 Refactoring <wyattscarpenter>
| |/
| * 2025-03-25 01:53 (Tue) 9ee603e Initial state <wyattscarpenter>
| * 2025-03-25 01:50 (Tue) 6c30b9d (origin/master, origin/HEAD) Update README.md: elaborate on project. <wyattscarpenter>
|/
* 2025-03-25 01:46 (Tue) 76bb797 Initial commit <wyattscarpenter>
```

# A third time?! 

What happens if I then rebase again?

