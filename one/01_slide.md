!SLIDE 
# SaltStack #

!SLIDE

## An alternative to puppet

!SLIDE 
## Why? 

!SLIDE

engagor-puppet is legacy, v2.8. Today at v4.7

!SLIDE bullets

* Ubuntu 12.04 => 16.04.
* Fix or start from scratch?

!SLIDE code

    @@@ ruby
    if $operatingsystem == 'debian' {
      include php54
    } else {
      include php53
    }

!SLIDE

That's how we detect we're on GCE.

!SLIDE

Not kidding :(

!SLIDE code small

    @@@ ruby

    class engagorweb {
      include engagorbase

      rsync::config { ["node"] }
      package { ["apache", ...]:
        ensure => present
      }
      class { "fail2banlocal": jails => ['haproxy'] }
      crons::config { ["engagor-web"] }
    }

!SLIDE bullets

* Hard to understand
* Hard to change
* deploy-puppet = #YOLO

!SLIDE

# So F@#$% Puppet

!SLIDE

# SaltStack <3 

!SLIDE bullets small

# Quick overview

* master -> minions pubsub over encrypted zeromq
* At the core it's remote command execution
* States = expressed as yaml + jinja
* Pillars = master data
* Grains = client side data (os, memory, ...)
* Reactive: act on events

!SLIDE code small

    @@@ sh 

    # matching hosts is cool

    # on minion id
    salt 'server501' ...
    salt 'server[56789]01' ...

    # using grains
    salt -C "-G@os:Debian" ...

    # using pillar data
    salt -I "elastic.cluster_name:mentions_1" 

    # batching
    salt -I "elastic.cluster_name:mentions_1" --batch-size 3 ...

!SLIDE

# DSL examples

!SLIDE code small

    @@@ yaml
    # copying a file
    /etc/logrotate.d/apache2:
      file.managed:
        - user: root
        - group: root
        - mode: 644
        - source: salt://files/apache/logrotate.cfg

!SLIDE code small
    @@@ yaml
    # dealing with big files, say extract a tarball
    /data/elasticsearch:
      archive.extracted:
        - source: http://packages.engagor.com/elasticsearch-1.7.1.tar.gz
        - source_hash: md5=45c2d87b47de8db162dbcb29cdd03fae
        - archive_format: tar
        - if_missing: /data/elasticsearch/bin/elasticsearch

!SLIDE code small  
    @@@ yaml
    # Install engagor code base
    git@github.com:engagor/engagor.git:
      git.latest:
        - target: /data/code/engagor
        - user: dev

!SLIDE 

cmd.run, service.running, pkg.installed
, pkgrepo.managed
, virtualenv.managed
, cron.present
, user.present
, ssh_auth.present
, ssh_known_hosts.present
, serverdensity_device.monitored
, glusterfs.created
, ...

!SLIDE code small

# top.sls

    @@@ yaml
    base:
      '*':
        - common

      'server501':
        - app.apache
        - app.haproxy
        - app.jenkins

      'server[56789]02':
        - app.elasticsearch
        - app.elasticsearch.mentions

!SLIDE code small
## salt/common/init.sls
    include:
      - common.backup
      - common.cron.all
      - common.dirs
      - common.dns
      - common.firewall
      - common.hosts
      - common.locales

## on disk:
    › ls salt/common -1
    backup.sls
    cron/
    dirs.sls
    dns.sls
    firewall.sls
    hosts.sls
    init.sls
    locales.sls

!SLIDE bullets small

## Super flexible

* grains: e.g. list databases so you can do "-G@db:engagor"
* pillars: store pillar data in a database, json or encrypted
* customize outputs: log in kibana, post in slack or on sonora
* reactor events: automate highstate when new minion joins


!SLIDE bullets small

# Testing
* run against a single node: salt 'server501' state.apply
* run a single state: salt 'server501' state.apply common.firewall
* docker env for local testing


!SLIDE

git@github.com:engagor/engagor-salt.git

!SLIDE

Questions?
