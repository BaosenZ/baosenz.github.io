---
title: "Setup GitHub Pages for Blog Using Chirpy Jekyll Theme"
date: 2024-12-28 12:00:00 +0000
categories: [Tech, Blog]
tags: [Tech, Github, Jekyll]
image: ../assets/blog_files/files_blog20241228/image-3.png
---


## 1 Introduction

This post is the guide of how to setup GitHub Pages for personal blog using Chirpy Jekyll Theme. I used Windows 11. If you used Windows 11, you can just follow along. After this finishing all the steps, here is what you will have:  
![alt text](../assets/blog_files/files_blog20241228/image-3.png)

## 2 Guides

### 2.1 GitHub Repo Setup

First, we need to setup the GitHub Repo following the [get-started tutorial by Chirpy](https://chirpy.cotes.page/posts/getting-started/). I used the recommended starter method. The image below is the steps:
![alt text](../assets/blog_files/files_blog20241228/image.png)

### 2.2 Local Blog Setup

Install Jekyll locally. I used Windows 11. I can directly use [documentation method posted by Jekyll](https://jekyllrb.com/docs/installation/windows/). RubyInstaller was used. Confirm the installation of Jekyll is complete by `jekyll -v`.   
Then, we git clone our repo locally:
```bash
git clone <repo-github.io-link>
```
If we open with VScode, you could see the folder structure like:
![alt text](../assets/blog_files/files_blog20241228/image-1.png)
We change the directory to the github.io link, and execute the jekyll:
```bash
cd <repo-github.io-link>
bundle exec jekyll s
```
Here is the output and you can visualize your blog in address: http://127.0.0.1:4000/:
![alt text](../assets/blog_files/files_blog20241228/image-2.png)

### 2.3 First Hello World Blog

Find out `_config.yml` file in main folder. Change the configurations there following the instructions in the file. We can change the `timezone`, `title`, `url`, `username`, `social`, `avatar`.  
Create our first post `yyyy-mm-dd-blog-title.md` in `_post` folder.  Put our image in `/assets/blog_files/files_blog19950505/sample-image.jpeg` folder. 

```markdown
---
title: "My First Chirpy Blog Post"
date: 1995-05-05 12:00:00 +0000
categories: [Test, Chirpy]
tags: [Test, Chirpy, Jekyll]
image: /assets/blog_files/files_blog19950505/sample-image.jpeg
---

Welcome to my first blog post using the Chirpy Jekyll theme! 🚀

## Getting Started

This is a test post to explore how Chirpy renders content. Below are some features you can use in your posts:

### Adding an Image
Here’s an image example (make sure the file exists in assets/img/posts/):
![image](/assets/blog_files/files_blog19950505/sample-image.jpeg)
```

### 2.4 GitHub Pages Setup and deploy
Go to the "GitHub repo > Settings > Pages". Setup the branch we want to deploy, usually "main" branch and save it.   
For the deploy, just type the following:
```bash
git add .
# git commit -m <message>, below is the example
git commit -m "first test blog"
git push
```
It will take a minute to deploy. But after it's done, you can go to your github pages to open your sites to check. You should see similar website shown in the first image. 

## 3 Reference
1. The youtube video and his post are good source to use chirpy Jekyll theme: https://technotim.live/posts/jekyll-docs-site/ and https://www.youtube.com/watch?v=F8iOU1ci19Q&ab_channel=TechnoTim
2. Here is the Chirpy documentation: https://chirpy.cotes.page/posts/getting-started/
3. GitHub Pages get started: https://docs.github.com/en/pages/quickstart
4. Install Jekyll on Windows: https://jekyllrb.com/docs/installation/windows/


