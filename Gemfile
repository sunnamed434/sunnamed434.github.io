# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll-theme-chirpy", "~> 7.5"

# google-protobuf < 4.28 does not support Ruby 3.4 (pulled in via sass-embedded → Jekyll)
gem "google-protobuf", ">= 4.28", "< 5.0"

gem "html-proofer", "~> 5.2", group: :test

platforms :windows, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.2.0", platforms: [:windows]

# Jekyll does not include this gem, so add it when running `jekyll serve` locally
gem "webrick", "~> 1.9"

gem "jekyll-compose", group: [:jekyll_plugins]
