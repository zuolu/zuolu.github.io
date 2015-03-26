---
layout: post
title:  "nginx unicorn部署rails项目"
date:   2015-03-26 11:41:00
categories: rails部署
comments: true
---

1. 安装nginx
sudo apt-get install nginx
命令会生成/etc/nginx/nginx.conf配置文件, /etc/nginx/sites-available和/etc/nginx/sites-enabled目录

nginx.conf配置文件:
{% highlight ruby %}
worker_processes 5;
user www-data; # for systems with a "nogroup"

pid /var/run/nginx.pid;
#error_log /path/to/nginx.error.log;

events {
  worker_connections 1024; # increase if you have lots of clients
  accept_mutex off; # "on" if nginx worker_processes > 1
}

http {
  include mime.types;

  default_type application/octet-stream;

  sendfile on;

  tcp_nopush on; # off may be better for *some* Comet/long-poll stuff
  tcp_nodelay off; # on may be better for some Comet/long-poll stuff

  gzip on;
  gzip_types application/json;

   include /etc/nginx/conf.d/*.conf;
   include /etc/nginx/sites-enabled/*;

}
{% endhighlight %}

sites-available/xiaocui文件配置:

{% highlight ruby %}
upstream app_server {
  server unix:/tmp/unicorn.xcuiblog.sock fail_timeout=0;
  #server 127.0.0.1:4201;
}

server {
  listen 80 default_server;
  server_name xcuiblog.com;
  #rewrite ^/(.*)$ http://xcuiblog.com permanent;
  try_files $uri/index.html $uri.html $uri @app;

  location @app {
    root /home/cuijunwei/PrivateCodes/xcuiblog/public;
    proxy_redirect     off;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header   X-Real-IP        $remote_addr;
    proxy_set_header Host $host:83;
    proxy_set_header   X-Forwarded-Host $host;

    proxy_pass http://app_server;
    proxy_set_header Via    "nginx";
  }
}
{% endhighlight %}
OK, nginx配置完毕.

配置unicorn:
gem install 'unicorn'
rails项目中Gemfile添加gem 'unicorn'
rails项目中创建文件config/unicorn.rb:
{% highlight ruby %}
worker_processes 1

app_root = File.expand_path("../..", __FILE__)
working_directory app_root

# Listen on fs socket for better performance
listen "/tmp/unicorn.xcuiblog.sock", :backlog => 64
#listen 4201, :tcp_nopush => false

# Nuke workers after 30 seconds instead of 60 seconds (the default)
timeout 30

# App PID
pid "#{app_root}/tmp/pids/unicorn.pid"

# By default, the Unicorn logger will write to stderr.
# Additionally, some applications/frameworks log to stderr or stdout,
# so prevent them from going to /dev/null when daemonized here:
stderr_path "#{app_root}/log/unicorn.stderr.log"
stdout_path "#{app_root}/log/unicorn.stdout.log"

# To save some memory and improve performance
preload_app true
GC.respond_to?(:copy_on_write_friendly=) and
  GC.copy_on_write_friendly = true

# Force the bundler gemfile environment variable to
# reference the Сapistrano "current" symlink
before_exec do |_|
  ENV["BUNDLE_GEMFILE"] = File.join(app_root, 'Gemfile')
end

before_fork do |server, worker|
  # 参考 http://unicorn.bogomips.org/SIGNALS.html
  # 使用USR2信号，以及在进程完成后用QUIT信号来实现无缝重启
  old_pid = app_root + '/tmp/pids/unicorn.pid.oldbin'
  if File.exists?(old_pid) && server.pid != old_pid
    begin
      Process.kill("QUIT", File.read(old_pid).to_i)
    rescue Errno::ENOENT, Errno::ESRCH
      # someone else did our job for us
    end
  end

  # the following is highly recomended for Rails + "preload_app true"
  # as there's no need for the master process to hold a connection
  defined?(ActiveRecord::Base) and
    ActiveRecord::Base.connection.disconnect!
end
{% endhighlight %}
然后执行: RAILS_ENV=production rake assets:precompile手动进行静态文件的编译

启动unicorn:
bundle exec unicorn -E production -c config/unicorn.rb

这样访问xcuiblog.com即可