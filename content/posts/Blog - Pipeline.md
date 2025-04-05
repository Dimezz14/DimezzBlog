
---
title: First Blog Post - 'The Pipeline'
date: 2025-04-03
draft: false
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
  

!![Image Description](/images/Pasted%20image%2020250403142357.png)

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
'cd static'

Use the ls command to make sure this folder is empty.
'ls'

Make a folder for your images
'mkdir images'

Once that is complete open your preferred code editor
My preferred code editor is Visual Studio Code

'code images.py'

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

!![Image Description](/images/Pasted%20image%2020250405114730.png)