# encoding: UTF-8

require 'rubygems'

require 'bundler/setup'
require 'bundler/cli'
require 'bundler/gem_tasks'

require 'rake/testtask'
require 'rake/notes/rake_task'
require 'rspec/core/rake_task'

require 'yard'

require 'rubocop/rake_task'

desc 'Run RuboCop'
RuboCop::RakeTask.new(:cop) do |task|
  task.patterns = ['lib/**/*.rb']
end

desc 'Run continuous integration test'
task :ci do
  Rake::Task['test:unit'].invoke
  unless ENV['TRAVIS'] == 'true' && ENV['TRAVIS_SECURE_ENV_VARS'] == 'false'
    Rake::Task['test:integration'].invoke
  end
  Rake::Task['test:cop'].invoke
end

namespace :gem do
  desc "Release gem version #{GoodData::VERSION} to rubygems"
  task :release do
    gem = "gooddata-#{GoodData::VERSION}.gem"

    puts "Building #{gem} ..."
    res = system('gem build ./gooddata.gemspec')
    next unless res

    puts "Pushing #{gem} ..."
    system("gem push #{gem}")
  end
end

namespace :hook do
  hook_path = File.join(File.dirname(__FILE__), '.git', 'hooks', 'pre-commit').to_s

  desc 'Installs git pre-commit hook running rubocop'
  task :install do
    if File.exist?(hook_path)
      puts 'Git pre-commit hook is already installed'
    else
      File.open(hook_path, 'w') do |file|
        file.write("#! /usr/bin/env bash\n")
        file.write("\n")
        file.write("rake cop\n")
      end
      system "chmod 755 #{hook_path}"
      puts 'Git commit hook was installed'
    end
  end

  desc 'Uninstalls git pre-commit hook'
  task :uninstall do
    res = File.exist?(hook_path)
    if res
      puts 'Uninstalling git pre-commit hook'
      system "rm #{hook_path}"
      puts 'Git pre-commit hook was uninstalled'
    else
      puts 'Git pre-commit hook is not installed'
    end
  end

  desc 'Checks if is git pre-commit hook installed'
  task :check do
    res = File.exist?(hook_path)
    if res
      puts 'Git pre-commit IS installed'
    else
      puts 'Git pre-commit IS NOT installed'
    end
  end
end

namespace :license do
  desc 'Show license report'
  task :info do
    Bundler::CLI.start(['exec', 'license_finder', '--decisions-file', 'dependency_decisions.yml'])
  end

  desc 'Generate licenses report - DEPENDENCIES.md'
  task :report do
    `bundle exec license_finder report --decisions-file dependency_decisions.yml --format=markdown > DEPENDENCIES.md`
  end

  desc 'Check if DEPENDENCIES.md is up to date'
  task :check do
    Rake::Task['license:report'].invoke
    res = `git diff --stat DEPENDENCIES.md`
    fail 'License check error' unless res.include?('1 file changed, 1 insertion(+), 1 deletion(-)')

    puts 'All licenses seem to be OK'
  end

  desc 'Add license header to each file'
  task :add do
    spec = Gem::Specification.load('gooddata.gemspec')
    license = File.readlines(File.expand_path('../LICENSE.rb', __FILE__))
    license << "\n"
    license_lines = license.length

    spec.files.each do |path|
      next if path == 'LICENSE.rb'
      next unless path.end_with?('.rb')

      puts "Processing #{path}"

      content = File.read(path)
      content_lines = content.lines

      update = content_lines.length < license_lines

      if update == false
        content_with_license = (license + content_lines[license_lines..-1]).join
        update = content != content_with_license
      end

      next unless update

      puts "Updating #{path}"

      if content_lines.length > 0 && content_lines[0].downcase.strip == '# encoding: utf-8'
        content_lines.slice!(0)
        content_lines.slice!(0) if content_lines[0] == "\n"
      end

      new_content = (license + content_lines).join
      File.open(path, 'w') { |file| file.write(new_content) }
    end
  end
end

RSpec::Core::RakeTask.new(:test)

namespace :test do
  desc 'Run unit tests'
  RSpec::Core::RakeTask.new(:unit) do |t|
    t.pattern = 'spec/unit/**/*.rb'
  end

  desc 'Run integration tests'
  RSpec::Core::RakeTask.new(:integration) do |t|
    t.pattern = 'spec/integration/**/*.rb'
  end

  desc 'Run legacy tests'
  RSpec::Core::RakeTask.new(:legacy) do |t|
    t.pattern = 'test/**/test_*.rb'
  end

  desc 'Run coding style tests'
  RSpec::Core::RakeTask.new(:cop) do
    Rake::Task['cop'].invoke
  end

  task :all => [:unit, :integration, :cop]
  task :ci => [:unit, :integration]
end

desc 'Run all tests'
task :test => 'test:all'

task :usage do
  puts 'No rake task specified, use rake -T to list them'
end

YARD::Rake::YardocTask.new

task :default => [:usage]
