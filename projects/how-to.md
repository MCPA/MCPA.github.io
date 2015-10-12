---
layout: page
title: Create a Write-up
permalink: /how-to/
author: "Leander"
date: YYYY-MM-DD HH:MM:SS
---

This page is designed to help members post write-ups using the Jekyll formatting mark-up. First things first. Clone the repo for this site:

<figure class="highlight"><pre><code class="language-text" data-lang="text">git clone https://github.com/MCPA/MCPA.github.io </code></pre></figure>

After you have the repo cloned you will be able to navigate to the "_posts" directory where you can add individual write-ups and then push them to the github site. Check out this <a name="Help Site" href="http://gitref.org/basic/">github help site</a> if you are unsure how to use a git repository.

<h3>Filename Template</h3>
Blog pages that are used to create write-ups must be created in the "_posts" directory and must follow a specific format for the file name. Where <code>YEAR</code> is a four-digit number, <code>MONTH</code> and <code>DAY</code> are both two-digit numbers, and <code>MARKUP</code> is the file extension representing the format used in the file.

<figure class="highlight"><pre><code class="language-text" data-lang="text">YEAR-MONTH-DAY-title.MARKUP </code></pre></figure>

 For example, the following are examples of valid post filenames:

<figure class="highlight"><pre><code class="language-text" data-lang="text">2011-12-31-new-years-eve-is-awesome.md
2012-09-12-how-to-write-a-blog.textile</code></pre></figure>

<h3>Content Formatting</h3>
The write-up blog must contain a header based on <a href="http://jekyllrb.com/docs/frontmatter/" hname="YAML Front Matter Link">YAML Front Matter</a>. In general each header should start out looking like the following:

<figure class="highlight"><pre><code class="language-text" data-lang="text">---
layout: post
title:  "Blogging Like a Hacker!!!"
author: HackerBob
date:   2015-09-27 21:56
categories: CSAW crypto Python C Perl
---
Place Content Here
</code></pre></figure>

Following the header, content formatting can be of just about any type such as, HTML or LaTEX. If you want to link to a specific image or file then you must upload it to the "_assets" directory. Additionally, if you are adding code blocks then there is also an option to enable syntax highlighting to make the code more readable.

<figure class="highlight"><pre><code class="language-text" data-lang="text">[My helpful screenshot]({{ site.url }}/assets/screenshot.jpg)
{{ "{% highlight perl linenos " }}%} code here {{ "{% endhighlight " }}%}</code></pre></figure>

When writing the solution begin with the location of where the challenge can be found if it's an online challenge or soft links to where you uploaded them on this github site.

Next, describe the steps you took to solve the challenge. This should include links where someone without your expertise can be able to find more if they want to research more. The general rule of thumb is that the reader should have knowledge of at least one high level programming language, a general concept of networking, and a general understanding of crypto.

We want to help someone understand not only the solution, but the what, how, and why you came to the solution you found. This will help develop the entire team.

<h3>Post It!</h3>
After you are done creating the initial filename following our formatting template and creating the content inside your write-up file, push your update to the github repo. The only files that you need to commit should be the _posts folder. After you have committed your updates Jekyll will handle the rest!

<figure class="highlight"><pre><code class="language-text" data-lang="text">git status #check and see what files you need to add
git add "list of files you are adding to the repo"
git commit -m "leave us all a message of what you did"
git push</code></pre></figure>
