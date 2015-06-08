FROM centos:7
MAINTAINER Linki <linki+docker.com@posteo.de>

# update the package list
RUN yum update -y

# install build-essential because we need to compile some native extensions
RUN yum install -y gcc-c++ make tar patch

# install zlib development headers for nokogiri to build
#
# http://www.nokogiri.org/tutorials/installing_nokogiri.html
#
RUN yum install -y zlib-devel

# we use a database for our backend, so let's add a couple of them
RUN yum install -y postgresql-devel mariadb mariadb-devel sqlite-devel

# add the nodejs repository to the package manager
#
# https://github.com/joyent/node/wiki/installing-node.js-via-package-manager#enterprise-linux-and-fedora
#
RUN curl -sL https://rpm.nodesource.com/setup | bash -

# newer versions of openproject use angular-js, so we want node and npm
#
# phantomjs also requires the bzip2 package, see
#   https://github.com/Medium/phantomjs/issues/267#issuecomment-67851777
#
RUN yum install -y nodejs bzip2

# install git in order to checkout gems defined by git urls
RUN yum install -y git

# install ruby 2.1 with development extensions
RUN yum install -y ruby ruby-devel

# install our beloved bundler
RUN gem install bundler

# install foreman to run our app server at the end
RUN gem install foreman

# clone openproject's default branch
RUN git clone --depth 1 https://github.com/opf/openproject.git /app

# set the working directory to the app
WORKDIR /app

# use a database.yml that takes its values from an environment variables
COPY ./files/database.yml /app/config/database.yml

# add some gems specific to this image build
COPY ./files/Gemfile.local /app/Gemfile.local

# create an unprivileged user that owns the code and will run the app server
RUN useradd -m app
RUN chown -R app:app /app

# switch to the openproject user from now on
USER app

# install all ruby dependencies via bundler into a local path
RUN bundle install --path ./vendor/bundle

# install all node dependencies via npm
RUN npm install

# precompile all assets
#
# openproject currently requires a database when compiling assets
# we fake one by providing an sqlite db location for this command only
#
# note, asset precompilation needs to be done in production, otherwise icon images are missing
#
RUN RAILS_ENV=production SECRET_TOKEN=foobar DATABASE_URL=sqlite3://db/ignore_me.sqlite3 \
  bundle exec rake assets:precompile

# expose the app server's port
EXPOSE 3000

# launch the rails server unless another command is given
CMD HOST=0.0.0.0 PORT=3000 foreman start web
