# frozen_string_literal: true

source 'https://rubygems.org'

require 'json'
require 'net/http'
versions = JSON.parse(Net::HTTP.get(URI('https://pages.github.com/versions.json')))

# 2nd arg is a ratchet to ensure it's at least current version when I last looked at it
gem 'github-pages', versions['github-pages'], '>= 226'
gem 'jekyll'
gem 'jekyll-feed'
gem 'jekyll-gist'
gem 'jekyll-toc'
gem 'jemoji'
gem 'redcarpet'
gem 'webrick', '>= 1.7'