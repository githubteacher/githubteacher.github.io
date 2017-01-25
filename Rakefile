#!/usr/bin/env rake

require 'html-proofer'

task :default => [:test]
desc 'Runs the tests!'
task :test do
  sh "bundle exec jekyll build"
  HTMLProofer.check_directory("./_site").run
end
