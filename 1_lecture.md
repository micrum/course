# 1. Першае Rails прыкладанне. Git, Bundler

## First Rails app

    $ rails new course-app

#### rails server

    $ cd course-app
    $ rails server

## Bundler

    $ bundle install

## Git

    $ git config --global user.name "Niels Bohr"
    $ git config --global user.email niels.bohr@atom1922.dk

#### .gitignore

    .idea

#### Repo

    $ git init

# add files to Git

    $ git add .

#### git commit

    $ git commit -m "Initial commit"

#### cancel not fixed chages
   
    $ git status
    $ git checkout -f

#### git push

   $ git remote add origin https://github.com/micrum/course-app.git
   $ git push -u origin master

#### git branches

    $ git checkout -b change-web-server
    $ git branch

*Gemfile*

    gem 'puma'
    
Ibstall puma and Merging:

    $ bundle
    $ git commit -a -m "Update web-server"
    $ git log
    $ git checkout master
    $ git merge change-web-server
    $ git branch -d change-web-server





[<< папярэдні занятак](0_lecture.md)
[наступны занятак >>](2_lecture.md)
