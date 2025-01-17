require "bundler/gem_tasks"
require "rake/testtask"

task :default => [:compile, :test]

namespace :version do
  desc "Bump version"
  task :bump do
    strscan_c_path = "ext/strscan/strscan.c"
    strscan_c = File.read(strscan_c_path).gsub(/STRSCAN_VERSION "(.+?)"/) do
      version = $1
      "STRSCAN_VERSION \"#{version.succ}\""
    end
    File.write(strscan_c_path, strscan_c)
  end
end

if RUBY_ENGINE == "jruby"
  require 'rake/javaextensiontask'
  Rake::JavaExtensionTask.new("strscan") do |ext|
    require 'maven/ruby/maven'
    ext.source_version = '1.8'
    ext.target_version = '1.8'
    ext.ext_dir = 'ext/java'
  end
else
  require 'rake/extensiontask'
  Rake::ExtensionTask.new("strscan")
end

desc "Run test"
task :test do
  ENV["RUBYOPT"] = "-Ilib -Itest/lib -rbundler/setup -rhelper"
  ruby("run-test.rb")
end

desc "Run benchmark"
task :benchmark do
  ruby("-S",
       "benchmark-driver",
       "benchmark/scan.yaml")
end

task :sync_tool do
  require 'fileutils'
  FileUtils.cp "../ruby/tool/lib/core_assertions.rb", "./test/lib"
  FileUtils.cp "../ruby/tool/lib/envutil.rb", "./test/lib"
  FileUtils.cp "../ruby/tool/lib/find_executable.rb", "./test/lib"
end

release_task = Rake.application["release"]
release_task.prerequisites.delete("build")
release_task.prerequisites.delete("release:rubygem_push")
release_task_comment = release_task.comment
if release_task_comment
  release_task.clear_comments
  release_task.comment = release_task_comment.gsub(/ and build.*$/, "")
end

desc "Push built gems"
task "push" do
  require "open-uri"
  helper = Bundler::GemHelper.instance
  gemspec = helper.gemspec
  name = gemspec.name
  version = gemspec.version.to_s
  pkg_dir = "pkg"
  mkdir_p(pkg_dir)
  ["", "-java"].each do |type|
    base_url = "https://github.com/ruby/#{name}/releases/download"
    url = URI("#{base_url}/v#{version}/#{name}-#{version}#{type}.gem")
    path = "#{pkg_dir}/#{File.basename(url.path)}"
    url.open do |input|
      File.open(path, "wb") do |output|
        IO.copy_stream(input, output)
      end
      helper.__send__(:rubygem_push, path)
    end
  end
end
