
I chose to try to write the Touist documentation using something
a little more confortable than the good-and-old latex. The language
is called Madoko, it is close to markdown (so very readable) and there
is an online editor (madoko.net/editor.html).

If you want to edit the .mdk file, you have multiple choices:

- (1. online madoko GUI and Github account)
  You first have to fork the touist project: https://github.com/touist/touist/fork
  Then, go to https://www.madoko.net/editor.html and log into your github account.
  Select the project freshly forked with
      "Open..." -> "Github" -> "yourgithubname/touist" -> "master" -> "docs/" ->
  Then, you can open the .mdk.
  To submit the changes, do "Synchronize...". Type a commit message that explains
  what you changed and press "Commit".
  Finally, go to your github fork https://github.com/yourgithubname/touist and
  open a new pull request against the https://github.com/touist/touist repo.

- (2. madoko GUI locally)
  Install madoko-local with `npm install -g madoko-local` and you run the madoko
  webserver with `madoko-local --port 8080 --run -l .` from the `docs` directory
  and you Open... Local Disk... and you select the .mdk file.
  To get your changes reviewed and accepted, fork the project, commit your changes
  and submit a pull request.

- (3. no GUI)
  Install madoko (not madoko-local): `npm install -g madoko`.
  Edit the `.mdk` file; you can then render the HTML or PDF with:
  * HTML: madoko reference-manual.mdk
  * PDF: madoko reference-manual.mdk --pdf
  You can then submit a pull request with your changes.

When submitting your pull-request, you will be able to check that the
`reference-manual.pdf` and `reference-manual.html` are looking as expected
through the `CircleCI` build. For example, if your PR is number 196,
you can browse the produced `reference-manual.{html,pdf}` here:

    https://circleci.com/gh/touist/touist/196#artifacts/containers/0


Thanks a lot for your help!!


About latex rendering and issues
================================

If you want to use madoko locally, you will need the full Texlive 2016 distrib;
if you use Texlive-basic (like me) and install tex packages with
`tlmgr install <package>`, you will need to install plenty of packages
error after error. In `circleci.yml` (at the root of the repository), you will
have an overview of all necessary ubuntu packages you can install using apt-get.

But a lot of errors can occur (I struggled a lot with latex errors). So what I
do to develop locally is that I use the "mathjax" mode when rendering the html,
which I can enable with the following metadata (at the top of the .mdk file):

    Math mode: mathjax

This means that you won't need latex for using madoko or madoko-local, and the
math figures will be rendered using javascript. CircleCI will take care of
turning on the statically-generated math pieces with the latex mode when
generating the final document (when the changes are pulled to the master branch).


NOTE: Do not put the HTML or PDF into github. The HTML and PDF files are
generated by the circle.yml on circleci.com and then published onto
    http://touist.github.io/
Changes will be published onto http://touist.github.io once the commits
are pulled into the master branch.


How are colored the Touist and Grammar
======================================

In the mdk file, we can read:

``` Touist
a and b
```

which is a block of code with the touist syntax coloring. The syntax
definition is contained in touist.json and grammar.json. They are syntax
definitions for Monarch. Everything necessary for understanding these
files is in https://microsoft.github.io/monaco-editor/monarch.html.


Known errors with madoko
========================

With the error (madoko 1.0.4, Texlive 2016):

    reference-manual: analyse and embed math images.
    error: cannot read: out/math/reference-manual-math-plain-1.svg
    fs.js:681
    return binding.rename(pathModule._makeLong(oldPath),
                    ^
This error has been reported on the madoko website:
http://madoko.codeplex.com/workitem/162

The actual errors doesn't show. But you can show the error with `madoko -vvvv`

    "dvisvgm" -n -b0.2pt -e -j -v3 -d3 -p1-40 -o "math/reference-manual-math-plain-%1p.svg" "reference-manual-math-plain.xdv"

    DVI error: DVI format 7 not supported

This error only happens with dvisvgm v1.15.1 (shipped with Texlive 2016), and does not
occur with dvisvgm 1.9.2 (shipped with Texlive 2015). This "Dvi format 7" error is
disscussed here: https://github.com/mgieseki/dvisvgm/issues/59


Ideas to improve madoko
=======================

## -mvalue:key should overwrite existing metadata declarations

When passing -mkey:value to madoko, for example to give the git version number, like

    madoko -mversion:`git describe --tags` reference-manual.mdk
because I want to set the git last tag dynamically in reference-manual.mdk:

    Title Note: Draft, `&date; (touist &version;)`

and the "&version;" is replaced with "v2.3.0-22-ga4450f3fa" for example.

When run in madoko-local, an error says that "version" is not known. My idea is to set
a "default" version number, for example:

    Version: "N/A"

but it won't work because the -mkey:value thing does not "replace" the existing metadata.

This behaviour could also help because I usually want to have

    Math mode: mathjax

enabled when I am locally working with madoko-local (I don't have the full latex
distrib). So to enable the static latex rendering when CircleCI generates the HTML,
I use the following parameter:

    madoko -mmath-mode:static reference-manual.mdk

But because `Math mode: mathjax` has precedence over the command-line parameter,
I cannot do that.


## Multiline tables

It would be great to be able to do multiline tables... For example with the ":"
syntax that continues the line (note that the new line with two trailing spaces
would not work...):

```
|-----------|------------------|
| Heading 1 | Centered 2       |
+-----------|:----------------:+
| Cell 1    | Cell 2           |
| Cell 1    | This cell        |
:           : continues on the :
:           : same line.       :
|-----------|------------------|
```

# Edit `reference-manual.mdk` in one click

It is already possible to open a document on madoko.net that is hosted on
the web with `?#url=http://...`, but the files surrounding it don't seem
to be loaded. What I propose is to enable the loading of these files.
Example: if I have the file

    https://github.com/touist/touist/tree/master/docs/reference-manual.mdk

and it uses the external colorizer `Colorizer: touist`, then madoko-local would
load the file

    https://github.com/touist/touist/tree/master/docs/touist.json

A simpler way would be for madoko-local to copy all the files that have the prefix

    https://github.com/touist/touist/tree/master/docs/

into the browser cache. But I have no idea how that would be possible to list
these files (easy if the document is hosted on a simple web server that has
direcoty listing enable (`Options +Indexes`), or if the document is on github
we can use the github API)
