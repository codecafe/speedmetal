# Install Gems
    jruby -S gem install bundler
    cd speedmetal/apps/rails3/hello_world
    jruby -S bundle install
    gem install bundler
    bundle install


# TorqueBox Setup

## Create TorqueBox deployment descriptor
    rm -f $JBOSS_HOME/server/default/deploy/*-knob.yml
    cat << EOF > $JBOSS_HOME/server/default/deploy/hello-knob.yml
    ---
    application:
      root: /home/ec2-user/speedmetal/apps/rails3/hello_world/
      env: production
    web:
      context: /
    EOF
## Start TorqueBox
    $JBOSS_HOME/bin/run.sh -b 0.0.0.0


# Passenger Setup

## Copy app to /tmp
Passenger needs the nginx workers that run as the 'nobody' user
to have access to the application.
    rm -rf /tmp/hello_world/
    cp -r speedmetal/apps/rails3/hello_world/ /tmp/

## Start Passenger
    cd /tmp/hello_world/
    passenger start -p 8080 -e production


# Thin Setup

## Start Thin
    cd speedmetal/apps/rails3/hello_world/
    thin -p 8080 -e production start


# Trinidad Setup

## Start Trinidad
    cd speedmetal/apps/rails3/hello_world/
    jruby -S trinidad -r -p 8080 -e production -t


# Unicorn Setup

## Write Config File
    cat << EOF > /tmp/unicorn.rb
    worker_processes N
    preload_app true
    timeout 30
    listen 8080, :backlog => 2048
    EOF

## Start Unicorn
    cd speedmetal/apps/rails3/hello_world/
    unicorn_rails -E production -c /tmp/unicorn.rb
