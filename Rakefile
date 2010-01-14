$:.unshift "lib" if File.directory? "lib"
require 'rcov/rcovtask'
require 'rcov/version'
require 'rake/testtask'
require 'rake/rdoctask'
require 'rake/gempackagetask'
require 'rake/clean'

# Use the specified rcov executable instead of the one in $PATH
# (this way we get a sort of informal functional test).
# This could also be specified from the command like, e.g.
#   rake rcov RCOVPATH=/path/to/myrcov
ENV["RCOVPATH"] = "bin/rcov"

# The following task is largely equivalent to:
# Rcov::RcovTask.new
desc "Create a cross-referenced code coverage report."
Rcov::RcovTask.new do |t|
  t.test_files = FileList['test/*_test.rb']
  t.ruby_opts << "-Ilib:ext/rcovrt" # in order to use this rcov
  t.rcov_opts << "--xrefs"  # comment to disable cross-references
  t.verbose = true
end

desc "Analyze code coverage for the FileStatistics class."
Rcov::RcovTask.new(:rcov_sourcefile) do |t|
  t.test_files = FileList['test/file_statistics_test.rb']
  t.verbose = true
  t.rcov_opts << "--test-unit-only"
  t.ruby_opts << "-Ilib:ext/rcovrt" # in order to use this rcov
  t.output_dir = "coverage.sourcefile"
end

Rcov::RcovTask.new(:rcov_ccanalyzer) do |t|
  t.test_files = FileList['test/code_coverage_analyzer_test.rb']
  t.verbose = true
  t.rcov_opts << "--test-unit-only"
  t.ruby_opts << "-Ilib:ext/rcovrt" # in order to use this rcov
  t.output_dir = "coverage.ccanalyzer"
end

desc "Run the unit tests with rcovrt."
if RUBY_PLATFORM == 'java'
  def java_classpath_arg # myriad of ways to discover JRuby classpath
    begin
      cpath  = Java::java.lang.System.getProperty('java.class.path').split(File::PATH_SEPARATOR)
      cpath += Java::java.lang.System.getProperty('sun.boot.class.path').split(File::PATH_SEPARATOR)
      jruby_cpath = cpath.compact.join(File::PATH_SEPARATOR)
    rescue => e
    end
    unless jruby_cpath
      jruby_cpath = ENV['JRUBY_PARENT_CLASSPATH'] || ENV['JRUBY_HOME'] &&
        FileList["#{ENV['JRUBY_HOME']}/lib/*.jar"].join(File::PATH_SEPARATOR)
    end
    jruby_cpath ? "-cp \"#{jruby_cpath}\"" : ""
  end

  Rake::TestTask.new(:test_rcovrt => ["lib/rcovrt.jar"]) do |t|
    t.libs << "lib"
    t.ruby_opts << "--debug"
    t.test_files = FileList['test/*_test.rb']
    t.verbose = true
  end

  file "lib/rcovrt.jar" => FileList["ext/java/**/*.java"] do |t|
    rm_f "lib/rcovrt.jar"
    mkdir_p "pkg/classes"
    sh "javac #{java_classpath_arg} -d pkg/classes #{t.prerequisites.join(' ')}"
    sh "jar cf #{t.name} -C pkg/classes ."
  end
else
    Rake::TestTask.new(:test_rcovrt => ["ext/rcovrt/rcovrt.so"]) do |t|
    system("cd ext/rcovrt && make clean && rm Makefile")
    t.libs << "ext/rcovrt"
    t.test_files = FileList['test/*_test.rb']
    t.verbose = true
  end
end

file "ext/rcovrt/rcovrt.so" => FileList["ext/rcovrt/*.c"] do
  ruby "setup.rb config"
  ruby "setup.rb setup"
end

desc "Run the unit tests in pure-Ruby mode ."
Rake::TestTask.new(:test_pure_ruby) do |t|
  t.libs << "ext/rcovrt"
  t.test_files = FileList['test/turn_off_rcovrt.rb', 'test/*_test.rb']
  t.verbose = true
end

desc "Run the unit tests"
task :test => [:test_rcovrt]

desc "install by setup.rb"
task :install do
  sh "sudo ruby setup.rb install"
end

task :default => :test

begin
  %w{sdoc sdoc-helpers rdiscount}.each { |name| gem name }
  require 'sdoc_helpers'
rescue LoadError => ex
  puts "sdoc support not enabled:"
  puts ex.inspect
end

require 'rake/rdoctask'
Rake::RDocTask.new do |rdoc|
  version = File.exist?('VERSION') ? File.read('VERSION') : ''
  rdoc.rdoc_dir = 'rdoc'
  rdoc.title = "rcov #{version}"
  rdoc.rdoc_files.include('README*')
  rdoc.rdoc_files.include('lib/**/*.rb')
end
