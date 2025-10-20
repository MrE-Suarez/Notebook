# Uploading files from the machine to a repo

>> Once the files are created...

1. **Intialize repo:** git init 
Makes the folder a Git repository (creates a hidden .git folder that stores history, branches, and settings)

*** Cycle
2. ***Snapshot files:*** git add <folderName> # or use . to include all
Selects the files to include in the next commit (specific folder or . for all)

*** Cycle
3. ***Save snapshot:***  git commit -m "Reason for commit note" -m "optional description"
Saves the files and changes locally (not yet uploaded to GitHub)

----- First time set up -----
Must: Create a new repo on GitHub with the same name as your local folder.

4. ***Link to remote:*** git remote add origin git@github.com:MrE-Suarez/demo-repo.git
Links the local repository to the one on GitHub.

    4.1 *Confirm remote:* git remote -v
    (Optional) Lists all linked remotes (for fetch/push)
-----End of first time set up -----

*** Cycle
5. ***Upload changes:*** git push origin main
Pushes commits to GitHub: push can be used to upload the files
(The branch can be main or master — check yours with git branch.)

    5.1 *Optional shortcut:* git push -u origin main
    Sets this branch as default for future pushes/pulls (-u = upstream).
    Next time you can just use git push or git pull.
    (Also creates the branch on GitHub if it doesn’t exist yet.)

Cycle: Repeat add → commit → push for every change made.