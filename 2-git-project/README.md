# INITIALIZING A REPOSITORY AND MAKING COMMITS

## WHAT IS GIT?

Git is a version control system that allows you to keep track of changes made in your code. It also helps you collaborate with other developer. Before Git, there were other technologies available that solves this problem and a good example is SVN. Git adopted a differench approach from that of SVN which allows developers to make their copies of the central repository, and that is why it is referred to as a Distributed Version Control System.

## INITIALIZING A GIT REPOSITORY
* Before we can initialize a git repository, we must have installed git on our computer. After installing, to initialize a repo. We follow the below steps;

* We open a terminal on our computer e.g Git bash; the default terminal after downloading and installing Git.

* We open the terminal and create a working folder or directory e.g DevOps folder using this command mkdir DevOps.

* After creating the folder or directory, we'll have to change or move into the working directory using this command `cd DevOps`.

* While inside the folder/directory, we use the git init command to initialize our repository.

![git 1](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/5fda4adb-2a44-4e7f-b841-fb5593095bb7)

## Making The First Commit
In the picture above, we successfully created our working directory and initialized a git repository. Now we will make our first commit. In git, commit simply means saving the changes (adding, modifying or deleting files or text) you made to your files. When we make a commit, git takes a snapshot of the current state of our repository and saves a copy in the .git folder inside our working directory.

Now, let's make our first commit by following the below steps;

* Inside our working directory, we create a new file called index.txt using this command touch index.txt.

* After creating, input any text in the text file.

* Then we add the changes to git staging area using this command `git add`.

* To commit the changes to git, we use this command git commit -m "Initial commit". The -m option/flag is used to add a commit message. The commit message provides context about the commit.

![git 2](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/b36022e6-d55b-4b79-b040-efba00a92bca)

## WORKING WITH BRANCHES
### working with branches
Git branches are a powerful tool that can help you to manage your development workflow more effectively. By using branches, you can experiment with new features, fix bugs, and collaborate with other developers without affecting the main codebase.
### Make your first branch
* to make new branch use `git checkout -b [branch name]`.
* The `-b flag` helps to create and change directly into the new branch.

![git 3](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/a4812ace-5074-4813-88fa-852dda253be5)

## Listing Branches
to show the list of the branches in the repository run this command `git branch`

![git 4](https://github.com/hammedakinwale/Darey.io-Linux-Project/assets/78992096/c49ab731-4f46-4e35-9cda-9bae2b508147)
