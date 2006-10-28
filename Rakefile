require 'rubygems'
require 'rake'
require 'rake/testtask'
require 'rake/gempackagetask'
require 'rake/rdoctask'
require 'rake/contrib/sshpublisher'

def announce(msg='')
  STDERR.puts msg
end

PKG_BUILD = ENV['PKG_BUILD'] ? '.' + ENV['PKG_BUILD'] : ''
PKG_NAME = 'namecase'
PKG_VERSION = '1.0.2' + PKG_BUILD
PKG_FILES = FileList["{doc,lib,test}/**/*"].exclude("rdoc").to_a

spec = Gem::Specification.new do |s|
  s.name      = PKG_NAME
  s.version   = PKG_VERSION
  s.author    = "Aaron Patterson"
  s.email     = "aaronp@rubyforge.org"
  s.homepage  = "#{PKG_NAME}.rubyforge.org"
  s.platform  = Gem::Platform::RUBY
  s.summary   = "NameCase is a Ruby implementation of Lingua::EN::NameCase, a library for converting strings to be properly cased."
  s.files     = Dir.glob("{bin,test,lib,doc}/**/*").delete_if {|item| item.include?(".svn") }
  s.require_path  = "lib"
  s.has_rdoc      = true
  s.extra_rdoc_files = ["README", "CHANGELOG", "LICENSE"]
  s.rdoc_options << "--main" << 'README' << "--title" << "'NameCase RDoc'"
  s.rubyforge_project = PKG_NAME
end

Rake::GemPackageTask.new(spec) do |p|
  p.gem_spec = spec
  p.need_tar = true
  p.need_zip = true
end

Rake::RDocTask.new do |p|
  p.main = "README"
  p.rdoc_dir = "doc"
  p.rdoc_files.include("README", "CHANGELOG", "LICENSE", "lib/**/*.rb")
  p.options << "--main" << 'README' << "--title" << "NameCase RDoc"
end

desc "Publish the API documentation"
task :pubrdoc => [ :rdoc ] do
  Rake::SshDirPublisher.new(
    "#{ENV['USER']}@rubyforge.org",
    "/var/www/gforge-projects/#{PKG_NAME}/",
    "doc" ).upload
end

task :update_version do
  announce "Updating Mechanize Version to #{PKG_VERSION}"
  File.open("lib/version.rb", "w") do |f|
    f.puts "# DO NOT EDIT"
    f.puts "# This file is auto-generated by build scripts"
    f.puts "class NameCase < String"
    f.puts "  Version = '#{PKG_VERSION}'"
    f.puts "end"
  end
  sh 'svn commit -m"updating version" lib/version.rb'
end

desc "Create a new release"
task :release => [ :clobber, :update_version, :package, :tag ] do
  announce 
  announce "**************************************************************"
  announce "* Release #{PKG_VERSION} Complete."
  announce "* Packages ready to upload."
  announce "**************************************************************"
  announce 
end

desc "Tag code"
Rake::Task.define_task("tag") do |p|
  baseurl = "svn+ssh://#{ENV['USER']}@rubyforge.org//var/svn/#{PKG_NAME}"
  sh "svn cp -m 'tagged #{ PKG_VERSION }' . #{ baseurl }/tags/REL-#{ PKG_VERSION }"
end

desc "Branch code"
Rake::Task.define_task("branch") do |p|
  baseurl = "svn+ssh://#{ENV['USER']}@rubyforge.org/var/svn/#{PKG_NAME}"
  sh "svn cp -m 'branched #{ PKG_VERSION }' #{baseurl}/trunk #{ baseurl }/branches/RB-#{ PKG_VERSION }"
end
