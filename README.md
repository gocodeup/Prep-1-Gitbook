# Prep 1 Gitbook

This is the control and presentation layer for our first phase prep work.

## Initial Setup

If you're just getting started writing a Gitbook, there's a handful of commandline tools you'll need to install, and only a couple of files to change before you begin adding content.

### Installing Utilities

Gitbook is written using [Node.js](http://nodejs.org), so first things first you'll need to install this utility. If you're on a Mac using Homebrew you can run the following command:

~~~bash
brew install node
~~~

Once you have node installed, install `gitbook-cli` using `npm` (depending on your setup, this may need to be done using `sudo`). This is the utility used to manage Gitbook's own dependencies and versions.

```bash
npm install -g gitbook-cli
```

The Gitbook CLI utility can automatically install the necessary version of Gitbook whenever we try to manipulate a particular book. This is done whether you're installing dependencies, serving the preview, or compiling the book for distribution. The Gitbook version is specified in `book/book.json` and is currently set to `~2.6`.

Now that you have the Gitbook software, you will need to install the plugins for this book. Although they are tracked in NPM, Gitbook is happier when it gets to manage those dependencies itself. To install them, run:

```
# Install dependencies for Gitbook stored in 'book' directory
gitbook install book
```

Finally, the deployment script is written in Ruby and requires a couple of gems to run. Most people will not need to worry about deploying the books, but if necessary you can install these using Bundler:

~~~bash
bundle install
~~~

### Setting up Submodules

In order to consolidate our curriculum content, we store it all in a separate repository and include it here as a submodule. Typically, this submodule is stored under `book/content`. If you have just cloned this repository, you will probably need to pull these files into your repo as well. In order to do that, run the following Git commands:

```bash
git submodule init
git submodule update
```

This will configure the submodule settings for your checkout and then clone the current commit of our curriculum content.

**Note:** The above process can be simplified by passing the flag `--recursive` to the `git clone` command:

```bash
git clone --recursive [repository-url] [directory]
```

### Repository File Structure

This repository contains some basic utilities and configuration options that I think should be useful for most of our books. Most of these you shouldn't need to modify but it's good to know what's going on.

- `.gitignore` &mdash; Default git ignore; ignores files generated by `gitbook` and managed by `npm` & `bundler`
- `.tm_properties` &mdash; Some handy options for editing Gitbooks in TextMate
- `s3-upload.rb` &mdash; Ruby script to push this book to Amazon S3; more on this below.
- `codeship.sh` &mdash; Shell script to install Node & Gitbook dependencies and compile the book.
- `Gemfile` & `Gemfile.lock`&mdash; Ruby gem settings for `s3-upload.rb`
- `package.json` &mdash; Node.js settings for Gitbook. Although it would be good to change these to reflect your project for the most part it isn't strictly necessary.
- `book` &mdash; This is where all the content of your Gitbook will go. Anything added to this directory will be included in the compiled book, and any markdown file will be rendered as HTML.
    - `.bookignore` &mdash; Files to be excluded from the compiled book; by default only ignores the `_book` directory `gitbook` uses for serving the dev site.
    - `.htpasswd` &mdash; Default passwords for use with [s3auth](http://www.s3auth.com). Includes the `codeup` and `codeupe-staff` users. (**Deprecated!**)
    - `book.json` &mdash; Properties and settings for your book. In particular, look at:
        - `title` & `description` &mdash; Your book's title and description, **do** modify these values
        - `plugins` &mdash; Some default Gitbook plugs we commonly use
        - `links` &mdash; Disables all the default sharing links and adds a couple of extra links to the sidebar for us
        - `variables` &mdash; Variables that can be used in Markdown. For example, the `curriculumName` variable is used to include different versions of exercises for different curriculums. For more information, see Gitbook's official [documentation](https://help.gitbook.com/format/templating.html#variables)
    - `content` &mdash; Curriculum content, included as a submodule of this repository
    - `cover.jpg` & `cover_small.jpg` &mdash; A simple default cover for your book.
    - `GLOSSARY.md` &mdash; An empty glossary file for your book, see the [documentation](https://help.gitbook.com/format/glossary.html)
    - `README.md` &mdash; An empty landing page; this will be the introduction for your book
    - `SUMMARY.md` &mdash; This is the file that defines your book's table of contents. It is an unordered list of links. Subsections can be created by adding child bullet points.

## Recommended Tools

GitHub's tools for editing Gitbooks are built for editing books hosted on their service. Because of this, they are not well suited for our workflow or curriculum. Instead, using the `gitbook` command line tool to serve the rendered output locally and using a plain text editor seems to work at least as well, and provides much more tailored experience. For a text editor, I'd recommend any one of the following:

- [Sublime Text](https://www.sublimetext.com) &mdash; Well, obviously
- [TextMate 2](http://macromates.com/download) &mdash; My preferred editor, has some additional facilities for editing Markdown
- [Mou](http://25.io/mou/) &mdash; Purpose built split pane Markdown editor. Has nice layout and can roughly approximate the CSS Gitbook uses (only one file at a time however)
- [MacDown](http://macdown.uranusjr.com) &mdash; Another split pane editor similar to Mou with some additional nice formatting support.

### Previewing

To view your Gitbook use the `gitbook` commandline tool. From within this directory run:

```bash
gitbook serve book
```

Then in your browser go to http://localhost:4000.

## Deployment

We use Amazon S3 to host the rendered Gitbook and [Codeship](https://codeship.com/) for continuous deployment. To aide in this process we've developed an `s3-upload.rb` script. This script will simply take the contents of a directory and upload them to an S3 bucket. These steps should be more or less automated so it is unlikely you will need to run this script by hand. Regardless, you can see its documentation by running `./s3-upload.rb -h`:

    Usage: s3-upload [options]
        -b, --bucket=BUCKET              S3 Bucket to deploy to (Required, default: $BUCKET)
        -a, --acl=ACL_NAME               S3 ACL to apply to all objects (Required, default: $BUCKET_ACL)
        -d, --dir=DIRECTORY              Directory to upload (Required)
        -k, --aws_key=KEY                AWS Upload Key (Required, default: $AWS_ACCESS_KEY_ID)
        -s, --aws_secret=SECRET          AWS Upload Secret (Required, default: $AWS_SECRET_ACCESS_KEY)
        -h, --help                       Display this help

Setting up Codeship takes a few steps.

1. This repository will need to be linked as a project in Codeship. This should be relatively straight forward through their interface.
1. Use `./codeship.sh` to set up the environment. There are no test commands for our Gitbooks.
1. Set up a new deployment branch, we typically use `production`. Use another custom script to deploy to S3:

    ```bash
    bundle install --without development
    ./s3-upload.rb -d build -b [your bucket name] -a [bucket-acl]
    ```

    Typically the bucket name is the same as the production domain name. The bucket ACL is typically either `public-read` or `authenticated-read`, depending on whether you are putting an authentication layer on top of this Gitbook.
1. You should set up `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` environment variables for authentication with Amazon.
1. Codeship will automatically set up a deployment key with in the repository. Unfortunately, because we are using submodules for most of our content this doesn't work for us.
    1. Go to this repository's settings and delete the Codeship deployment key.
    1. Go to the Codeship project general settings and copy the SSH public key.
    1. Log in to the `instructors@codeup.com` GitHub account, go to the account settings, and add the Codeship key there. Now both this repository and its submodules can be accessed by Codeship.

Obviously, there are some steps necessary for configuring Amazon S3 along with all this. That...is beyond the scope of this document however.
