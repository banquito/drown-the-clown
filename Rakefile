#Starts by default in production mode
#Later, the setting can be changed by calling:
# {App}.set :environment, :production | :development | etc
# {App}.reconfigure()

ENV['RACK_ENV'] = 'production'
require './drown'
class Application < App
end

def build_assets output_dir
  app = Application.new.helpers
  puts 'Resolving assets...'
  assets = app.get_assets
  assets_dir = File.join output_dir , app.settings.assets_prefix
  assets.each do |asset|
    path = File.join assets_dir, asset.logical_path
    puts "Building '#{path}'"
    asset.write_to path
  end

  puts 'Copying Resources'
  resources = File.join Application.resources_path, '.'
  FileUtils.cp_r resources, assets_dir
end

def build_file output_dir,file_name, file_content
  output = File.join output_dir, file_name
  puts "Building '#{output}'"
  File.write output, file_content
end

def build_index output_dir
  Application.set :recompile_views, true
  build_file output_dir, 'index.html', Application.new.helpers.index
end

def build_changelog output_dir
  build_file output_dir, 'changelog.html', Application.new.helpers.changelog
end

def build_htaccess output_dir
  build_file output_dir,'.htaccess', Application.new.helpers.htaccess
end

def build_service output_dir
  build_file output_dir,'service.php', Application.new.helpers.service
end

def build output_dir, production
  if production
    Application.set :use_key, :production
  end

  puts "Building static application in '#{output_dir}'"

  FileUtils.rm_r output_dir, :force => true
  FileUtils.mkdir_p output_dir
  build_assets output_dir
  build_index output_dir
  build_changelog output_dir
  build_htaccess output_dir
  build_service output_dir
end

task :build_assets, [:output_dir] do |task, args|
  output_dir = args[:output_dir] || 'build'
  puts "Building assets application in '#{output_dir}'"
  assets_dir = File.join output_dir, Application.settings.assets_prefix
  FileUtils.rm_r assets_dir, :force => true
  build_assets output_dir
end

desc 'Builds the static application, by default, using development keys.'
task :build, [:output_dir, :use_parse_production] do |task, args|
  output_dir = args[:output_dir] || 'build'
  build output_dir,args[:use_parse_production]
end

desc 'Builds the static application using production keys.'
task :build_production, [:output_dir] do |task, args|
  output_dir = args[:output_dir] || 'production'
  build output_dir, true
  #adding .htaccess file only for production
  build_htaccess output_dir
end

desc 'Builds the static web application (dashboard).'
task :build_web, [:output_dir,:production] do |task, args|
  output_dir = args[:output_dir] || 'parse/public'
  Application.set :index,          :web
  Application.set :main_js,        'clothespin-web.js'
  Application.set :main_css,       'clothespin-web.css'
  # Application.set :resources_path, 'assets/resources-web'
  build output_dir, args[:production]
end


def get_notes version
  %x{sed -n '/#{version}/,$p' changelog.md | sed '/^$/,$d'}
end
