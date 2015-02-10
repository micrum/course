# Installing Ruby on Rails on Ubuntu

### Installing RVM and Ruby

[Ruby Version Manager](https://rvm.io/) - RVM is a command-line tool which allows you to easily install, manage, and work with multiple ruby environments from interpreters to sets of gems.

Install cUrl and RVM:

    $ sudo apt-get install curl
    $ curl -L https://get.rvm.io | bash -s

Load RVM and install dependencies:
    
    $ source /home/ubuntu/.rvm/scripts/rvm
    $ rvm requirements

Install additional dependencies:

    $ sudo apt-get -y install build-essential openssl libxml2-dev libxslt-dev subversion

Check out available Ruby interpreter versions:

    $ rvm list known

Install Ruby and set OpenSSL location:

    $ rvm install 2.2.0 --with-openssl-dir=$HOME/.rvm/usr

Create a default gemset:

    $ rvm use 2.2.0@courseby --create --default

### Installing Rails

[RubyGems](https://rubygems.org/) is a package manager for Ruby projects, and there are many useful libraries (including Rails) available as Ruby gems. RVM includes RubyGems by default. 

Install Rails:

    $ gem install rails --version 4.2.0

Install JavaScript runtime using PPA:

    $ sudo apt-add-repository ppa:chris-lea/node.js
    $ sudo apt-get update
    $ sudo apt-get install nodejs

### Installing Git

[Git](http://git-scm.com/) is a free and open source distributed version control system designed to handle everything from small to very large projects with speed and efficiency.

Install Git:

    $ sudo apt-get install git
