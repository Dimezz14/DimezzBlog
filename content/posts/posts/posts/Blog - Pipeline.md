
---
title: First Blog Post - 'The Pipeline'
date: 2025-04-03
lastmod: 2025-04-05
author: Joshua E
tags:
- DimezzInTech
- blog
---


## ***Obsidian - Note application***
- This will be the application that we use to sync our blog posts with Hugo
- Obsidian Link : https://obsidian.md
- Note*:  Only the Notes created within the posts folder will appear on the website.

#### *Task 1 - Create Posts Folder*
- Create a folder within Obsidian labeled posts.
- Make sure the locate where the Obsidian directory is located.
  

![[Pasted image 20250403142357.png]]

## ***Setting up Hugo***
- This is will be the framework we use for building the blog website.

#### *Task 1 - Hugo Installation*
For this blog I am using and Mac and all installations used the 'Homebrew' commands.
- Installing Git: https://github.com/git-guides/install-git
	- Homebrew Command: '*brew install git*'
- Installing Go: https://go.dev/doc/install
	- Homebrew Command: '*brew install go*'
- Installing Hugo: https://gohugo.io/installation/
	- Homebrew Command : '*brew install hugo*'


#### *Task 2 - Check Versioning *
The Check Versioning Tasks is used to make sure all installations were successful.
- Git Command: '*git version*'
- Go Command: '*go version*'
- Hugo Command: '*hugo version*'


#### *Task 3 - Create New Site with Hugo *

```
##Create Hugo Site

hugo new site sitename
cd sitename
```

##### **Download a Hugo Theme**
Hugo Themes Link: https://themes.gohugo.io
- Most themes will have instructions on how to download. The option I chose were themes with submodule.

```
## Initialize a git repository (Make sure you're in your Hugo site directory)

git init

## Set up your global username and email parameters for git

git config --global user.name "Name"
git config --global user.email "Email Address"

## Install a theme (the theme I selected was Newsroom)
git submodule add https://github.com/onweru/newsroom.git themes/newsroom

## The newsroom module wants you to edit the hugo.toml file

Use nano hugo.toml to open and edit the hugo.toml file

Within the file make sure to set theme as 
'theme = ["github.com/onweru/newsroom"]'
then run
hugo mod init yourWebsiteName && hugo mod get -u
```

##### **Test Hugo**

```
## Verify Hugo works with your theme by running this command 

hugo server -t themename

```

#### ***Task 4 - Syncing Obsidian to Hugo***
```
rsync -av --delete "sourcepath" "destinationpath"
```

##### Adding Frontmatter

```
---
title: 'blogtitle'
date: '2025-04-03'
draft: false
tags:
  - 'tag1'
  - 'tag2'
---

```


##### Transferring Images from Obsidian to Hugo

In your terminal switch to the static folder
- *`'cd static'`*

Use the ls command to make sure this folder is empty.
- *`'ls'`*

Make a folder for your images
- *`'mkdir images'`*

Once that is complete open your preferred code editor
My preferred code editor is Visual Studio Code

- *`'code images.py'`*

In your images.py file paste the code below

```
import os
import re
import shutil

# Paths
posts_dir = "/Users/joshuaellis/Documents/DimezzBlog/content/posts/"
attachments_dir = "/Users/joshuaellis/Obsidian/Attachments/"
static_images_dir = "/Users/joshuaellis/Documents/DimezzBlog/static/images/"

# Step 1: Process each markdown file in the posts directory
for filename in os.listdir(posts_dir):
    if filename.endswith(".md"):
        filepath = os.path.join(posts_dir, filename)
        
        with open(filepath, "r") as file:
            content = file.read()
        
        # Step 2: Find all image links in the format ![Image Description](/images/Pasted%20image%20...%20.png)
        images = re.findall(r'\[\[([^]]*\.png)\]\]', content)
        
        # Step 3: Replace image links and ensure URLs are correctly formatted
        for image in images:
            # Prepare the Markdown-compatible link with %20 replacing spaces
            markdown_image = f"![Image Description](/images/{image.replace(' ', '%20')})"
            content = content.replace(f"[[{image}]]", markdown_image)
            
            # Step 4: Copy the image to the Hugo static/images directory if it exists
            image_source = os.path.join(attachments_dir, image)
            if os.path.exists(image_source):
                shutil.copy(image_source, static_images_dir)

        # Step 5: Write the updated content back to the markdown file
        with open(filepath, "w") as file:
            file.write(content)

print("Markdown files processed and images copied successfully.")
```

Make sure the following directories match your paths

![[Pasted image 20250405114730.png]]


#### Task 5 - Setting up GitHub

- Create a Github Account
- Then create a new repository for your blog posts
  
  ###### Authentication - SSH Key
```
## Generate an SSH key
## In your terminal run command below
'cd ~/.ssh'

## Run the ls command to check the files within
'ls'

## To create a SSH Key run command below & follow the terminal prompts
'ssh-keygen -t rsa -b 4096 -C "your_email@example.com'

## Run the ls command to check for the new file you created (ie. 'ida_rsa & ida_rsa.pub')
'ls'

## We will be using the .pub file to get the output for your public key
cat ida_rsa.pub
```

Copy that ssh key into GitHub by going to settings and clicking SSH & GPG Keys
Now using your terminal in the .ssh directory use this command to connect with Github.

- '*`ssh -T git@github.com'`*

Now lets move back into our blog folder to push the changes we made. Use the following commands to push to Github.

```
git remote add origin git@github.com:'Github Account Name'/'New Repository Name'

## The hugo command will start building the site
hugo

git add .

git commit -m "First blog commit"

git push --set-upstream origin main

```

##Creating a new branch called Hostinger to push all of our changes
echo "Deploying to GitHub Hostinger..."
git subtree split --prefix public -b hostinger-deploy
git push origin hostinger-deploy:hostinger --force
git branch -D hostinger-deploy


# The Mega Script


```
#!/bin/bash
set -euo pipefail

# Change to the script's directory
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
cd "$SCRIPT_DIR"

# Set variables for Obsidian to Hugo copy
sourcePath="/Users/joshuaellis/Obsidian/posts"
destinationPath="/Users/joshuaellis/Documents/DimezzBlog/content/posts"

# Set GitHub Repo
myrepo="reponame"

# Check for required commands
for cmd in git rsync python3 hugo; do
    if ! command -v $cmd &> /dev/null; then
        echo "$cmd is not installed or not in PATH."
        exit 1
    fi
done

# Step 1: Check if Git is initialized, and initialize if necessary
if [ ! -d ".git" ]; then
    echo "Initializing Git repository..."
    git init
    git remote add origin $myrepo
else
    echo "Git repository already initialized."
    if ! git remote | grep -q 'origin'; then
        echo "Adding remote origin..."
        git remote add origin $myrepo
    fi
fi

# Step 2: Sync posts from Obsidian to Hugo content folder using rsync
echo "Syncing posts from Obsidian..."

if [ ! -d "$sourcePath" ]; then
    echo "Source path does not exist: $sourcePath"
    exit 1
fi

if [ ! -d "$destinationPath" ]; then
    echo "Destination path does not exist: $destinationPath"
    exit 1
fi

rsync -av --delete "$sourcePath" "$destinationPath"

# Step 3: Process Markdown files with Python script to handle image links
echo "Processing image links in Markdown files..."
if [ ! -f "images.py" ]; then
    echo "Python script images.py not found."
    exit 1
fi

if ! python3 images.py; then
    echo "Failed to process image links."
    exit 1
fi

# Step 4: Build the Hugo site
echo "Building the Hugo site..."
if ! hugo; then
    echo "Hugo build failed."
    exit 1
fi

# Step 5: Add changes to Git
echo "Staging changes for Git..."
if git diff --quiet && git diff --cached --quiet; then
    echo "No changes to stage."
else
    git add .
fi

# Step 6: Commit changes with a dynamic message
commit_message="New Blog Post on $(date +'%Y-%m-%d %H:%M:%S')"
if git diff --cached --quiet; then
    echo "No changes to commit."
else
    echo "Committing changes..."
    git commit -m "$commit_message"
fi

# Step 7: Push all changes to the main branch
echo "Deploying to GitHub Main..."
if ! git push origin main; then
    echo "Failed to push to main branch."
    exit 1
fi

# Step 8: Push the public folder to the hostinger branch using subtree split and force push
echo "Deploying to GitHub Hostinger..."
if git branch --list | grep -q 'hostinger-deploy'; then
    git branch -D hostinger-deploy
fi

if ! git subtree split --prefix public -b hostinger-deploy; then
    echo "Subtree split failed."
    exit 1
fi

if ! git push origin hostinger-deploy:hostinger --force; then
    echo "Failed to push to hostinger branch."
    git branch -D hostinger-deploy
    exit 1
fi

git branch -D hostinger-deploy

echo "All done! Site synced, processed, committed, built, and deployed."```