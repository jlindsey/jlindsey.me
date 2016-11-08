require 'rake/clean'

CLEAN << 'build'

desc 'Build site'
task :build do
  sh 'bundle', 'exec', 'middleman', 'build'
end

desc 'Deploy site'
task deploy: :build do
  sh 'aws', 's3', 'sync', '--delete', 'build', 's3://jlindsey-me'
end
