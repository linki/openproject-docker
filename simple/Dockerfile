FROM ubuntu:14.04
MAINTAINER Linki <linki+docker.com@posteo.de>

# update the package list
RUN apt-get update

# install build-essential because we need to compile some native extensions
RUN apt-get -y install build-essential

# we use a database for our backend, so let's add a couple of them
RUN apt-get -y install libpq-dev libmysql++-dev sqlite3 libsqlite3-dev

# newer versions of openproject use angular-js, so we want node and npm
RUN apt-get -y install nodejs npm

# allow the nodejs binary to be called via node (this should be fixed somehow)
RUN ln -s nodejs /usr/bin/node

# install git in order to checkout openproject's source
RUN apt-get -y install git-core

# install ruby 2.1
#
# the following installs ruby 2.1 via the apt package manager. unfortunately,
# the default packages still only contain ruby 1.9, so we use brightbox's one.
RUN apt-get install -y software-properties-common python-software-properties
RUN apt-add-repository -y ppa:brightbox/ruby-ng
RUN apt-get update

# install ruby 2.1 with development extensions
RUN apt-get install -y ruby2.1 ruby2.1-dev

# clone openproject's default branch
RUN git clone --depth 1 https://github.com/opf/openproject.git /usr/src/openproject

# set the working directory to the app
WORKDIR /usr/src/openproject

# use a database.yml that takes its values from an environment variables
COPY ./files/database.yml /usr/src/openproject/config/database.yml

# add some gems specific to this image build
COPY ./files/Gemfile.local /usr/src/openproject/Gemfile.local

# install our beloved bundler
RUN gem install bundler

# install all ruby dependencies via bundler
RUN bundle install

# allow bower to run as root
RUN sed -i -e "s/bower install/bower install --allow-root/g" /usr/src/openproject/frontend/package.json

# install all node dependencies via npm
RUN npm install --unsafe-perm

# precompile all assets
#
# openproject currently requires a database when compiling assets
# we fake one by providing a sqlite db location for this command only
RUN DATABASE_URL=sqlite3://db/ignore_me.sqlite3 bundle exec rake assets:precompile

# tell docker that we intend to mount something here
VOLUME /usr/src/openproject/files

# expose the app server's port
EXPOSE 3000

# launch the rails server unless another command is given
CMD bundle exec rails server --port 3000
