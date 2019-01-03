---
layout: post
title: Website with Jekyll
---
This website is genereted using Jekyll. The [Step-By-Step-How-Two](https://jekyllrb.com/docs/step-by-step/01-setup/) is great but I missed some informations which I documented here.

## Preperation

### Ruby

At first install [Ruby](https://www.ruby-lang.org/en/). On Arch-Linux: 

``` bash
sudo pacman -S ruby
```

### Jekyll

The following command will install [Jekyll](https://jekyllrb.com/) in your user directory (*~/.gem/ruby/*).

``` bash
gem install --user-install jekyll bundler
```

You probably want to add the ruby bin folder to your search path by adding following line to your "run commands" file of choice. (*~/.bashrc*, *~/.zshrc*, ...).

``` bash
export PATH=$PATH:~/.gem/ruby/2.5.0/bin
```
Adapt the version of ruby accordingly to your installed version.

## Create a new Website

Now you are ready to start working on your website. I would suggest to instruct Jekyll to create a dummy website for you with:

``` bash
jekyll new <path_to_project_folder>
```
This has the advantage that jekyll will create a base directory structure and most importand create some config files for you.

Unfortunately this call fails on my system, because Jekyll tries to install some additional ruby gems and complains about missing super user rights. I decided not to install on the system level, therefore I had to install the missing gems by hand:

``` bash
gem install minima jekyll-feed
```

The following command builds the webiste under *\<path_to_project_folder>/_site*:

``` bash
cd <path_to_project_folder>
jekyll build
```

Jekyll has an build in webserver which can be started with:

``` bash
cd <path_to_project_folder>
jekyll serve
```

The website can be accessed via [http://localhost:4000](http://localhost:4000).
If one of the source files in *\<path_to_project_folder>* changes, then the webserver immediately updates the website in *\<path_to_project_folder>/_site*:

## Writing and Configuring a Website

A good step by step tutorial to write a website is provided on the [Jekyll Website](https://jekyllrb.com/docs/step-by-step/01-setup/). The next sections cover some additional topics not described in the step by step tutorial.

### Front Matter

Allways inlude a Front Matter even if you don't have to declare anything. Otherwise the Liquid statements in the documents will not be evaluated.

### Syntax Highlighting

In Markdown you can highlight code by fence it with three back-ticks.

~~~
```c
int main(int argc, char **argv);
```

```bash
export HELLO="Hello world!";
echo $HELLO;
```
~~~

The default highlighter of Jekyll is [Rouge](http://rouge.jneen.net/) which is compatible with [Pygments](http://pygments.org/). Rouge enrich the code with additional HTML tags and corresponding CSS classes. [Pygmentize](http://pygments.org/docs/cmdline/) can be used to generate according CSS definitions.
```bash
sudo pacman -S pygmentize
```
Use Pygmantize to create a style which is called "vim":

```bash
pygmentize -S vim -f html > syntax.css 
```

Alternatively you can download the [css files](https://github.com/richleland/pygments-css).

### Automatic Deployment using Git
For automatic deployment create a git remote the target machine.
change2
