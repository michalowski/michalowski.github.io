source "https://rubygems.org"

# Core Jekyll Setup
gem "jekyll", "~> 4.3.3"
gem "webrick" # Required for local serving on modern Ruby versions
gem "minimal-mistakes-jekyll"

# Missing Standard Libraries (Crucial for Ruby 3.x / 4.x compatibility)
gem "csv"
gem "base64"
gem "bigdecimal"
gem "ostruct"
gem "logger"
gem "mutex_m"

# Platform Specific Windows Gems
gem "tzinfo-data"
gem "wdm", ">= 0.1.0" if Gem.win_platform?

# Jekyll Plugins Workspace
group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-gist"
  gem "jekyll-feed"
  gem "jemoji"
  gem "jekyll-include-cache"
  gem "jekyll-algolia"
end
