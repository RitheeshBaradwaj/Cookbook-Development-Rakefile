task default: [:clean, :style, :test, :release]

desc 'Removes any policy lock files present, berks lockfile, etc.'
task :clean do
  %w(
    .kitchen
    metadata.json
    policies/*.lock.json
  ).each { |f| FileUtils.rm_rf(Dir.glob(f)) }
end

desc 'Run foodcritic and cookstyle on this cookbook'
task style: 'style:all'

namespace :style do
  # Cookstyle
  begin
    require 'cookstyle'
    require 'rubocop/rake_task'

    RuboCop::RakeTask.new(:cookstyle) do |task|
      # If we are in CI mode then add formatter options
      task.options.concat %w(
        --require rubocop/formatter/checkstyle_formatter
        --format RuboCop::Formatter::CheckstyleFormatter
        -o reports/xml/checkstyle-result.xml
      ) if ENV['CI']
    end
  rescue => exception
    puts ">>> Gem load error: #{e}, omitting style:cookstyle" unless ENV['CI']
  end

  # Load foodcritic
  begin
    require 'foodcritic'

    desc 'Run foodcritic style checks'
    FoodCritic::Rake::LintTask.new(:foodcritic) do |task|
      task.options = {
      fail_tags: ['any'],
      progress: true,
      }
    end
  rescue LoadError
    puts ">>> Gem load error: #{e}, omitting style:foodcritic" unless ENV['CI']
  end
  task all: [:cookstyle, :foodcritic]
end

desc 'Run unit and functional tests'
task test: 'test:all'
namespace :test do
  begin
    require 'rspec/core/rake_task'
    desc 'Run ChefSpec unit tests'
    RSpec::Core::RakeTask.new(:unit) do |t|
      t.rspec_opts = ENV['CI'] ? '--format RspecJunitFormatter --out rspec.xml' : '--color --format progress'
      t.pattern = 'test/unit/**{,/*/**}/*_spec.rb'
    end
  rescue
    puts ">>> Gem load error: #{e}, omitting tests:unit" unless ENV['CI']
  end

  begin
    require 'kitchen/rake_tasks'
    desc 'Run kitchen integration tests'
    Kitchen::RakeTasks.new
  rescue StandardError => e
    puts ">>> Kitchen error: #{e}, omitting #{task.name}" unless ENV['CI']
  end
  namespace :kitchen do
    desc 'Destroys all active kitchen resources'
    task :destroy do
      sh 'kitchen destroy'
    end
  end

  task all: ['test:unit', 'test:kitchen:all']
end
