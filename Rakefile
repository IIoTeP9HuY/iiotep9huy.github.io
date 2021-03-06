require 'rubygems'
require 'rake'
require 'rdoc'
require 'date'
require 'yaml'
require 'tmpdir'
require 'jekyll'
require 'rbconfig'
require 'fileutils'

# Load the configuration file.
CONFIG = YAML.load_file("_config.yml")

# Get and parse the date.
DATE = Time.now.strftime("%Y-%m-%d")
TIME = Time.now.strftime("%H:%M:%S")
POST_TIME = DATE + ' ' + TIME

# Directories.
POSTS = "_posts"
DRAFTS = "_drafts"
SITE = "_site"

# == Helpers ===================================================================

# Execute a system command.
def execute(command)
  system "#{command}"
end

# Chech the title.
def check_title(title)
  if title.nil? or title.empty?
    raise "Please add a title to your file."
  end
end

# Transform the filename and date to a slug.
def transform_to_slug(title, extension)
  characters = /("|'|!|\?|:|\s\z)/
  whitespace = /\s/
  "#{title.gsub(characters,"").gsub(whitespace,"-").downcase}.#{extension}"
end

# Read the template file.
def read_file(template)
  File.read(template)
end

# Save the file with the title in the YAML front matter.
def write_file(content, title, directory, filename)
  parsed_content = "#{content.sub("title:", "title: \"#{title}\"")}"
  parsed_content = "#{parsed_content.sub("date:", "date: #{POST_TIME}")}"
  File.write("#{directory}/#{filename}", parsed_content)
  puts "#{filename} was created in '#{directory}'."
end

# Create the file with the slug and open the default editor.
def create_file(directory, filename, content, title, editor)
  FileUtils.mkdir(directory) unless File.exists?(directory)
  if File.exists?("#{directory}/#{filename}")
    raise "The file already exists."
  else
    write_file(content, title, directory, filename)
    if editor && !editor.nil?
      sleep 1
      execute("#{editor} #{directory}/#{filename}")
    end
  end
end

# Get the "open" command.
def open_command
  if RbConfig::CONFIG["host_os"] =~ /mswin|mingw/
    "start"
  elsif RbConfig::CONFIG["host_os"] =~ /darwin/
    "open"
  else
    "xdg-open"
  end
end

# == Tasks =====================================================================

desc "Generate blog files"
task :generate do
  Jekyll::Site.new(Jekyll.configuration({
    "source"      => ".",
    "destination" => "_site"
  })).process
end


desc "Generate and publish blog to gh-pages"
task :publish => [:generate] do
  Dir.mktmpdir do |tmp|
    system "git push origin master"
    system "mv _site/* #{tmp}"
    system "git checkout gh-pages"
    system "rm -rf *"
    system "mv #{tmp}/* ."
    message = "Site updated at #{Time.now.utc}"
    system "git add ."
    system "git commit --allow-empty -m #{message.shellescape}"
    system "git push origin gh-pages"
    system "git checkout master"
    system "echo yolo"
  end
end

# rake post["Title"]
desc "Create a post in _posts"
task :post, :title do |t, args|
  title = args[:title]
  template = CONFIG["post"]["template"]
  extension = CONFIG["post"]["extension"]
  editor = CONFIG["editor"]
  check_title(title)
  filename = "#{DATE}-#{transform_to_slug(title, extension)}"
  content = read_file(template)
  create_file(POSTS, filename, content, title, editor)
end

task :default => :publish
