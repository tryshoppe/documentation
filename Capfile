set :application, "support"
set :repository, "git://github.com/tryshoppe/documentation.git"
set :branch, "master"
set :deploy_to, "/opt/rubyapps/shoppe-website/docs"
role :app, "tryshoppe.com"

namespace :deploy do
  desc 'Deploy the latest revision of the application'
  task :default do
    update_code
  end

  task :update_code, :roles => [:app, :storage] do
    run "cd #{deploy_to} && git branch -d rollback && git branch rollback"
    run "cd #{deploy_to} && git fetch origin && git reset --hard origin/#{fetch(:branch)}"
  end

  desc 'Setup the repository on the remote server for the first time'
  task :setup, :roles => [:app, :storage] do
    run "rm -rf #{deploy_to}"
    run "git clone -n #{fetch(:repository)} #{deploy_to} --branch #{fetch(:branch)}"
    run "cd #{deploy_to} && git branch rollback && git checkout -b deploy && git branch -d #{fetch(:branch)}"
    update_code
  end
end

