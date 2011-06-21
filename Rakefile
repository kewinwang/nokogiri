# -*- ruby -*-

require 'rubygems'
gem 'hoe', '>= 2.1.0'
require 'hoe'

windows = RUBY_PLATFORM =~ /(mswin|mingw)/i
java    = RUBY_PLATFORM =~ /java/

GENERATED_PARSER    = "lib/nokogiri/css/parser.rb"
GENERATED_TOKENIZER = "lib/nokogiri/css/tokenizer.rb"
CROSS_DIR           = File.join(File.dirname(__FILE__), 'tmp', 'cross')

Hoe.plugin :debugging
Hoe.plugin :git
Hoe.plugin :gemspec

HOE = Hoe.spec 'nokogiri' do
  developer 'Aaron Patterson', 'aaronp@rubyforge.org'
  developer 'Mike Dalessio',   'mike.dalessio@gmail.com'
  developer 'Yoko Harada',     'yokolet@gmail.com'

  self.readme_file       = ['README',    ENV['HLANG'], 'rdoc'].compact.join('.')
  self.history_file      = ['CHANGELOG', ENV['HLANG'], 'rdoc'].compact.join('.')
  self.extra_rdoc_files  = FileList['*.rdoc','ext/nokogiri/*.c']
  self.clean_globs = [
    'lib/nokogiri/*.{o,so,bundle,a,log,dll}',
    'lib/nokogiri/nokogiri.rb',
    'lib/nokogiri/1.{8,9}',
    GENERATED_PARSER,
    GENERATED_TOKENIZER,
    CROSS_DIR
  ]

  extra_dev_deps << ["racc", '>= 0']
  extra_dev_deps << ["rexical", '>= 0']
  extra_dev_deps << ["rake-compiler", '>= 0']
  extra_dev_deps << ["minitest", "~> 2.2.2"]

  if java
    self.spec_extras = { :platform => 'java' }
    self.need_tar = false # these will be built broken
    self.need_zip = false
  else
    self.spec_extras = {
      :extensions => ["ext/nokogiri/extconf.rb"],
      :required_ruby_version => '>= 1.8.7'
    }
  end

  self.testlib = :minitest
end

Hoe.add_include_dirs '.'

require 'tasks/nokogiri.org'

gem 'rake-compiler', '>= 0.4.1'
if java
  require "rake/javaextensiontask"
  Rake::JavaExtensionTask.new("nokogiri", HOE.spec) do |ext|
    jruby_home = RbConfig::CONFIG['prefix']
    ext.ext_dir = 'ext/java'
    ext.lib_dir = 'lib/nokogiri'
    jars = ["#{jruby_home}/lib/jruby.jar"] + FileList['lib/*.jar']
    ext.classpath = jars.map { |x| File.expand_path x }.join ':'
  end

  gem_build_path = File.join 'pkg', HOE.spec.full_name

  task gem_build_path => [:compile] do
    cp 'lib/nokogiri/nokogiri.jar', File.join(gem_build_path, 'lib', 'nokogiri')
    HOE.spec.files += ['lib/nokogiri/nokogiri.jar']
  end
else
  require "rake/extensiontask"
  Rake::ExtensionTask.new("nokogiri", HOE.spec) do |ext|
    ext.lib_dir = File.join(*['lib', 'nokogiri', ENV['FAT_DIR']].compact)

    ext.config_options << ENV['EXTOPTS']
    ext.cross_compile   = true
    ext.cross_platform = ["x86-mingw32", "x86-mswin32-60"]
    ext.cross_config_options <<
    "--with-xml2-include=#{File.join(CROSS_DIR, 'include', 'libxml2')}"
    ext.cross_config_options <<
    "--with-xml2-lib=#{File.join(CROSS_DIR, 'lib')}"
    ext.cross_config_options << "--with-iconv-dir=#{CROSS_DIR}"
    ext.cross_config_options << "--with-xslt-dir=#{CROSS_DIR}"
    ext.cross_config_options << "--with-zlib-dir=#{CROSS_DIR}"
  end
end

desc "Generate css/parser.y and css/tokenizer.rex"
task 'generate' => [ GENERATED_PARSER, GENERATED_TOKENIZER ]

file GENERATED_PARSER => "lib/nokogiri/css/parser.y" do |t|
  racc = RbConfig::CONFIG['target_os'] =~ /mswin32/ ? '' : `which racc`.strip
  racc = "#{::RbConfig::CONFIG['bindir']}/racc" if racc.empty?
  sh "#{racc} -l -o #{t.name} #{t.prerequisites.first}"
end

file GENERATED_TOKENIZER => "lib/nokogiri/css/tokenizer.rex" do |t|
  sh "rex --independent -o #{t.name} #{t.prerequisites.first}"
end

task 'gem:spec' => 'generate' if Rake::Task.task_defined?("gem:spec")

require 'tasks/test'
begin
  require 'tasks/cross_compile'
rescue RuntimeError => e
  warn "WARNING: Could not perform some cross-compiling: #{e}"
end

desc "set environment variables to build and/or test with debug options"
task :debug do
  ENV['NOKOGIRI_DEBUG'] = "true"
  ENV['CFLAGS'] ||= ""
  ENV['CFLAGS'] += " -DDEBUG"
end

# Only do this on unix, since we can't build on windows
unless windows
  [:compile, :check_manifest].each do |task_name|
    Rake::Task[task_name].prerequisites << GENERATED_PARSER
    Rake::Task[task_name].prerequisites << GENERATED_TOKENIZER
  end

  Rake::Task[:test].prerequisites << :compile
  Rake::Task[:test].prerequisites << :check_extra_deps unless java
  if Hoe.plugins.include?(:debugging)
    ['valgrind', 'valgrind:mem', 'valgrind:mem0'].each do |task_name|
      Rake::Task["test:#{task_name}"].prerequisites << :compile
    end
  end
end

# vim: syntax=Ruby
