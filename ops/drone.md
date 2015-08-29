# build

## vagrantfile

```ruby
Vagrant.configure(2) do |config|
  config.vm.box = "centos7-base"
  # public network to minimize config steps for testing
  config.vm.network "public_network"
end
```

### vagrant setup

1. `sudo yum update`
2. `sudo yum install docker`
3. `sudo systemctl start docker.service`
  1. in the event docker gives you backtalk about devmapper...
  2. remove and reinstall docker ([no, seriously](http://www.mogumagu.com/wp/wordpress/?p=1715))
2. [install docker-compose](https://docs.docker.com/compose/install/) 
3. `git clone https://github.com/drone/drone.git` into home directory
4. `ip addr` from prompt to get the ip vagrant negotiated with dhcp

## ~/docker-compose.yml
```yml
app:
  build: /home/vagrant/drone
  privileged: true
  volumes:
    /var/run/docker.sock:/var/run/docker.sock
    /var/lib/drone:/var/lib/drone
  ports:
    - "8080:80"
  environment:
    DRONE_GITHUB_SECRET: "<snipped>"
    DRONE_GITHUB_CLIENT: "<snipped>"
```

### docker-compose setup

1. [create an oauth application on github](https://github.com/settings/applications/) (or [any other repo provider](http://readme.drone.io/setup/config/github/))
2. copy the keys into the above YAML environment tags

## hnl.io/.drone.yml
```yml
image: ruby:2.2.3
env:
  - ENV=test
  - RAILS_ENV=test
script:
  - sed -i '2s/.*/exit 0/' /usr/sbin/policy-rc.d
  - echo "deb http://http.debian.net/debian jessie-backports main" >> /etc/apt/sources.list.d/backports.list
  - apt-get update
  - apt-get install -y apt-utils postgresql socat curl
  - cp config/database.drone.yml config/database.yml
  - echo "local all all trust" > /etc/postgresql/9.4/main/pg_hba.conf
  - echo "host all all localhost trust" >> /etc/postgresql/9.4/main/pg_hba.conf
  - /etc/init.d/postgresql reload
  - su -c "createdb hnl_io_test" postgres
  - apt-get install -y curl
  - curl --location https://deb.nodesource.com/setup_0.12 | bash -
  - apt-get install -y nodejs
  - apt-get install -y build-essential
  - bundle install
  - bundle exec rake db:schema:load
  - bundle exec rake test
services:
  - postgres
```

# other notes

- [vagrant network exposure types](https://stackoverflow.com/questions/23497855/unable-to-connect-to-vagrant-private-network-from-host)
- [leaned-out drone-docker image I couldn't make work first time around](https://hub.docker.com/r/mattgruter/drone/~/dockerfile/)
- [on docker-compose workflows](http://blog.bananacoding.com/blog/development-workflow-using-docker-and-docker-compose)
- [.drone.yml reference](https://github.com/drone/drone/blob/v0.2.1/README.md#builds)
- [postgres access conf](http://www.postgresql.org/docs/9.4/static/auth-pg-hba-conf.html)
- [installed service starting rules override](http://sharadchhetri.com/2015/02/21/prevent-starting-service-after-package-installation-on-ubuntu-debian/)
- [debian backports install](http://backports.debian.org/Instructions/)
- [drone ci setup, general](http://jipiboily.com/2014/from-zero-to-fully-working-ci-server-in-less-than-10-minutes-with-drone-docker/)
