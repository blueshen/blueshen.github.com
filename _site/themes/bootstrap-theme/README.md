# Bootstrap theme

## Prerequisites

Please check that you have a recent version of [compass](http://compass-style.org/) installed in octopress' bundle
(see Gemfile.lock in your octopress directory and run bundle update if necessary), otherwise, you might get errors
similar to those reported in issue #7. Compass version 0.12.1 is known to work.

## Installation

     $ git clone git://github.com/bkutil/bootstrap-theme.git bootstrap-theme

Copy the bootstrap-theme into your blog's octopress .theme directory:

     $ cp -R bootstrap-theme $MY_OCTOBLOG/.themes/bootstrap

Install the theme and generate site:

     $ rake install['bootstrap']
     $ rake generate

## Code snippet colors

Theme utilizes the solarized color scheme for code snippets. By default, the
bootstrap variant is selected, but light/dark colors can be used by setting
the $solarized variable in sass/syntax/\_higlight.scss.

## Patches welcome!

This is a first draft only. Any ideas, suggestions or improvements are welcome.

## Demo

Latest code from master branch is running at this [demo site](http://bootstrap-theme.kutilovi.cz).
