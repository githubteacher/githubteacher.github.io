#!/usr/bin/env rake

require 'html/proofer'
require 'html/pipeline'
require 'pathname'
require 'fileutils'
require 'open-uri'

task :default => [:test]
desc 'Runs the tests!'
task :test do
  ignored_links = []
  # ignore links to dotcom, since they're all authenticated
  ignored_links.push(%r{https?://github.com/})
  ignored_links.push(%r{\.githubapp\.com})
  ignored_links.push(/halp\.githubapp\.com/)
  ignored_links.push(/github\.app\.box\.com/)
  # ignore links to private GitHub Chrome Web Store extensions
  ignored_links.push(/chrome\.google\.com\/webstore\/detail\/github-team-notifications\/kodolpncoeenablbaffkhokfmhgiakik/)
  ignored_links.push(/chrome\.google\.com\/webstore\/detail\/halp-chrome\/jgejieaojhlfjmkgokhcdibbcfmcmcfm\/related/)
  # ignore links to linkedin.com, LinkedIn does not allow direct access to its URLs, one has to use LinkedIn's API instead
  ignored_links.push(%r{https?://(www.)linkedin.com})

  # fetch the latest enterprise number straight from the Help site
  LATEST_ENT_VERSION = open('https://help.github.com/').read.match(%r{enterprise/(.+?)/user})[1]

  href_swap = {
    # pretend any .md link is really a .html link
    /\.md/ => '.html',
    # rewrite Enterprise links to not rely on client-side redirects
    %r{/enterprise/admin/} => "/enterprise/#{LATEST_ENT_VERSION}/admin/",
    %r{/enterprise/user/} => "/enterprise/#{LATEST_ENT_VERSION}/user/"
  }

  # ignore poorly named image that can't be changed because no one knows the f.cl.ly login
  ignored_urls = [%r{f.cl.ly/.+?Screen%20Shot}]

  begin
    # make an out dir
    FileUtils.mkdir_p('tmp')

    pipeline = HTML::Pipeline.new [
      HTML::Pipeline::MarkdownFilter,
      HTML::Pipeline::TableOfContentsFilter
    ], :gfm => true

    # iterate over files, and generate HTML from Markdown
    Dir.glob('./docs/**/*') do |doc|
      dirpath = File.join('tmp', Pathname.new(File.dirname(doc)).cleanpath)
      FileUtils.mkdir_p(dirpath)

      if File.extname(doc) == '.md'
        filename = File.basename(doc).sub('.md', '.html')
        contents = File.read(doc)
        result = pipeline.call(contents)

        File.open(File.join(dirpath, filename), 'w') { |file| file.write(result[:output].to_s) }
      elsif File.file?(doc) # for asset images and the like
        FileUtils.copy(doc, dirpath)
      end
    end

    # test your tmp dir!
    HTML::Proofer.new('./tmp', {
                                :empty_alt_ignore => true,
                                :url_ignore  => ignored_urls,
                                :href_ignore => ignored_links,
                                :href_swap => href_swap,
                                :connecttimeout => 600,
                                :typhoeus => { :ssl_verifypeer => false, :ssl_verifyhost => 0, :followlocation => true }
                               }).run
  ensure
    FileUtils.rm_rf('tmp')
  end
end
