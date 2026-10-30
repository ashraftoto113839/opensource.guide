# Development runtime for the Jekyll site.
# No application source is copied in: the repo is bind-mounted at /app at run time,
# so edits are picked up by `jekyll serve --watch`. Only Ruby gems are installed here.
FROM ruby:2.6-buster

# Debian buster is archived, so point apt at the archive mirror.
RUN printf "deb http://archive.debian.org/debian buster main\ndeb http://archive.debian.org/debian-security buster/updates main\n" > /etc/apt/sources.list \
 && apt-get -o Acquire::Check-Valid-Until=false update -qq \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y -qq --no-install-recommends \
      libxml2-dev libxslt1-dev zlib1g-dev cmake curl ca-certificates \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Gem dependencies only (Gemfile.lock is from the github-pages 191 era, hence Ruby 2.6).
COPY Gemfile Gemfile.lock ./
RUN bundle config --global frozen 1 \
 && bundle install --jobs 4 --retry 3

EXPOSE 4000
