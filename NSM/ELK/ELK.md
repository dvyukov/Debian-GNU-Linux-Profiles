
#  Deployment of Bro/Suricata/Snort/ELK on Debian 9 GNU/Linux

1.  [Overview](#orgf8750f5)
    1.  [Configuration](#orgedcabb8)
        1.  [Topology](#org89c16b3)
        2.  [Bro](#org314a36d)
        3.  [ELK](#org1fb3593)
        4.  [Suricata](#org41a11e3)
        5.  [Snort](#orgb352335)
2.  [Adding Patterns for ELk](#org085730d)
    1.  [Bro](#org3f78268)
        1.  [More information](#orgcd09d14)
    2.  [Suricata & Snort](#org4322c5e)
        1.  [More Information](#org653ee21)
3.  [Adding a dashboard of data type](#orgddc8f71)
    1.  [Bro](#org9e3bb50)



<a id="orgf8750f5"></a>

# Overview

Nowadays, security analyst who are too busy to collect data through reading long tutorials and tedious processes. In this document that you don't need to learn each open source platform, just enough knowledge to be self-sufficient.If you’re a security analyst responsible for investigating alerts,  researching data, or responding to incidents then this document will be very useful and easy to help you building your own investigation platform.


<a id="orgedcabb8"></a>

## Configuration


<a id="org89c16b3"></a>

### Topology

Demo test:

![img](img/DemoTest-topology.png)


<a id="org314a36d"></a>

### Bro

First, review the [installing bro on Debian](https://github.com/hardenedlinux/Debian-GNU-Linux-Profiles/blob/master/docs/ids/bro-ids.md) Page.

-   configure Bro JSON logging 
    Add to this line `@load tuning/json-logs` to `/usr/local/bro/share/bro/site/local.bro`

1.  openssh

    how to configure a Bro cluster <https://www.bro.org/sphinx/cluster/index.html> learn more information. 
    
    1.  SSH login without password
    
    Step 1: Create public and private keys using ssh-key-gen on manager-host, you need that make sure that login in on `root` user. that's important for you.
    Step 2: Copy the public key to sensor-host using ssh-copy-id.
    other steps, you know how to do that.
    
    1.  configure your worker(s) in `/usr/local/bro/etc/node.cfg`:
    
        [manager]
        type=manager
        host=192.168.1.8 #manager host
         #
        [proxy-1]
        type=proxy
        host=192.168.1.8 #manager host
        #
        [worker-1]
        type=worker
        host=192.168.1.10 #sensor host
        interface=ens192
    
    3.Then start up a Bro instance:
    
    [BroControl] > start
                 > deploy
                 > status  //Reconfirm everything looks like good.
    manager      manager 192.168.1.8      running   3929   22 Dec 18:30:05
    proxy-1      proxy   192.168.1.8      running   3971   22 Dec 18:30:07
    worker-1     worker  192.168.1.10     running   13662  18 Dec 12:13:14

2.  Syslog

    Syslog configs are default stored in `/etc/syslog/syslog.conf`
    Syslog for your system following the instructions here. The following commands will category the source data of the bro:
    
        ## Lines to add to /etc/syslog-ng/syslog-ng.conf
        source s_bro_conn { file("/usr/local/bro/logs/current/conn.log" flags(no-parse) program_override("bro_conn")); };
        source s_bro_http { file("/usr/local/bro/logs/current/http.log" flags(no-parse) program_override("bro_http")); };
        source s_bro_dns { file("/usr/local/bro/logs/current/dns.log" flags(no-parse) program_override("bro_dns")); };
        source s_bro_files { file("/usr/local/bro/logs/current/files.log" flags(no-parse) program_override("bro_files")); };
        source s_bro_dhcp { file("/usr/local/bro/logs/current/dhcp.log" flags(no-parse) program_override("bro_dhcp")); };
        source s_bro_weird { file("/usr/local/bro/logs/current/weird.log" flags(no-parse) program_override("bro_weird")); };
        source s_bro_tunnels { file("/usr/local/bro/logs/current/tunnel.log" flags(no-parse) program_override("bro_tunnels")); };
        source s_bro_syslog { file("/usr/local/bro/logs/current/syslog.log" flags(no-parse) program_override("bro_syslog")); };
        source s_bro_ftp { file("/usr/local/bro/logs/current/ftp.log" flags(no-parse) program_override("bro_ftp")); };
        source s_bro_notice { file("/usr/local/bro/logs/current/notice.log" flags(no-parse) program_override("bro_notice")); };
        source s_bro_smtp { file("/usr/local/bro/logs/current/smtp.log" flags(no-parse) program_override("bro_smtp")); };
        source s_bro_smtp_entities { file("/usr/local/bro/logs/current/smtp_entities.log" flags(no-parse) program_override("bro_smtp_entities")); };
        source s_bro_ssl { file("/usr/local/bro/logs/current/ssl.log" flags(no-parse) program_override("bro_ssl")); };
        source s_bro_software { file("/usr/local/bro/logs/current/software.log" flags(no-parse) program_override("bro_software")); };
        source s_bro_irc { file("/usr/local/bro/logs/current/irc.log" flags(no-parse) program_override("bro_irc")); };
        source s_bro_ssh { file("/usr/local/bro/logs/current/ssh.log" flags(no-parse) program_override("bro_ssh")); };
        source s_bro_intel { file("/usr/local/bro/logs/current/intel.log" flags(no-parse) program_override("bro_intel")); };
        source s_bro_x509 { file("/usr/local/bro/logs/current/x509.log" flags(no-parse) program_override("bro_x509")); };
        source s_bro_snmp { file("/usr/local/bro/logs/current/snmp.log" flags(no-parse) program_override("bro_snmp")); };
        source s_bro_radius { file("/usr/local/bro/logs/current/radius.log" flags(no-parse) program_override("bro_radius")); };
        source s_bro_mysql { file("/usr/local/bro/logs/current/mysql.log" flags(no-parse) program_override("bro_mysql")); };
        source s_bro_kerberos { file("/usr/local/bro/logs/current/kerberos.log" flags(no-parse) program_override("bro_kerberos")); };
        source s_bro_rdp { file("/usr/local/bro/logs/current/rdp.log" flags(no-parse) program_override("bro_rdp")); };
        source s_bro_pe { file("/usr/local/bro/logs/current/pe.log" flags(no-parse) program_override("bro_pe")); };
        source s_bro_sip { file("/usr/local/bro/logs/current/sip.log" flags(no-parse) program_override("bro_sip")); };
        source s_bro_smb_mapping { file("/usr/local/bro/logs/current/smb_mapping.log" flags(no-parse) program_override("bro_smb_mapping")); };
        source s_bro_smb_files { file("/usr/local/bro/logs/current/smb_files.log" flags(no-parse) program_override("bro_smb_files")); };
        source s_bro_ntlm { file("/usr/local/bro/logs/current/ntlm.log" flags(no-parse) program_override("bro_ntlm")); };
        source s_bro_dce_rpc { file("/usr/local/bro/logs/current/dce_rpc.log" flags(no-parse) program_override("bro_dce_rpc")); };
        source s_bro_modbus { file("/usr/local/bro/logs/current/modbus.log" flags(no-parse) program_override("bro_modbus")); };
        source s_bro_dnp3 { file("/usr/local/bro/logs/current/dnp3.log" flags(no-parse) program_override("bro_dnp3")); };
        source s_bro_rfb { file("/usr/local/bro/logs/current/rfb.log" flags(no-parse) program_override("bro_rfb")); };
        
        destination d_bro { network("192.168.1.8" port(1000)); }; #manager host
        
        log {
            source(s_bro_conn);
            source(s_bro_http);
            source(s_bro_dns);
            source(s_bro_weird);
            source(s_bro_tunnels);
            source(s_bro_syslog);
            source(s_bro_ftp);
            source(s_bro_files);
            source(s_bro_dhcp);
            source(s_bro_notice);
            source(s_bro_smtp);
            source(s_bro_smtp_entities);
            source(s_bro_ssl);
            source(s_bro_irc);
            source(s_bro_software);
            source(s_bro_ssh);
            source(s_bro_smb_mapping);
            source(s_bro_smb_files);
            source(s_bro_ntlm);
            source(s_bro_dce_rpc);
            source(s_bro_intel);
            source(s_bro_x509);
            source(s_bro_snmp);
            source(s_bro_radius);
            source(s_bro_mysql);
            source(s_bro_kerberos);
            source(s_bro_rdp);
            source(s_bro_pe);
            source(s_bro_sip);
            source(s_bro_modbus);
            source(s_bro_dnp3);
            source(s_bro_rfb);
            log { destination(d_bro); };
        };

3.  logstrash

     They are consumed by syslog-ng and stored in ELK.
    the configuring file in `/etc/logstash/conf.d/`, create `bro.conf` at here and add following command to this conf file.
    
        if you did install ELK, just skiped here.
    
        input {
            syslog {
                port => "1000" #destination port
            }
        }
        
        filter {
            json {
                source => "message"
            }
        
            mutate {
                remove_field => ["message"]
                rename => {
                    "id.orig_h" => "srcip"
                    "id.orig_p" => "srcport"
                    "id.resp_h" => "dstip"
                    "id.resp_p" => "dstport"
                    "orig_bytes" => "src_bytes"
                    "resp_bytes" => "dst_bytes"
                }
            }
        
            geoip {
                source => "srcip"
                target => "geoip_src"
            }
        
            geoip {
                source => "dstip"
                target => "geoip_dst"
            }
        }
        
        output {
        
            elasticsearch {
                hosts => ["localhost:9200"]
                index => "bro"
            }
        }


<a id="org1fb3593"></a>

### ELK

1.  Installing ELK

        Download and install ELK for your system following the instructions here.
         sudo apt-get install openjdk-8-jre
         sudo dpkg -i elasticsearch-6.0.0.deb
         sudo /bin/systemctl daemon-reload
         sudo /bin/systemctl enable elasticsearch.service
         sudo systemctl start elasticsearch.service 
    
    `Other tools's` (kibana/logstrash) installed in a pretty similar process for that
    we're going to go website to the download page for logstash and others
    Finally, Checking your feedback information, we should have log stash installed correctly environment and it looks like
    everything got done or no errors.
    `curl -XGET localhost:9200`
    
     we're going to use the template for ELK that based on Security onion
     <https://github.com/Security-Onion-Solutions/elastic-test>
    Modify this  option  `config.reload.automatic: false` false to true which is going to enable this automatic configuration, After that we are going to go down here below it remove this interval to 3 table to default setting which is saying that every 3 seconds log says `config.reload.interval: 3s`
    Until this time we were finished all process for Logstash, restart serif again ~~
    
    <table>
    
    
    <colgroup>
    <col  class="org-left">
    
    <col  class="org-left">
    </colgroup>
    <thead>
    <tr>
    <th scope="col" class="org-left">Elasticserach</th>
    <th scope="col" class="org-left">&#xa0;</th>
    </tr>
    </thead>
    
    <tbody>
    <tr>
    <td class="org-left">Search</td>
    <td class="org-left">inverted indexs</td>
    </tr>
    
    
    <tr>
    <td class="org-left">structured data</td>
    <td class="org-left">Json</td>
    </tr>
    
    
    <tr>
    <td class="org-left">components</td>
    <td class="org-left">document, value, indes, types</td>
    </tr>
    
    
    <tr>
    <td class="org-left">&#xa0;</td>
    <td class="org-left">&#xa0;</td>
    </tr>
    </tbody>
    </table>


<a id="org41a11e3"></a>

### Suricata

-   Download and install Suricata for your system following the instructions here.

The default installation path is `/usr/local/bin/`, use the default configuration in `/usr/local/etc/suricata/` and will output to `/usr/local/var/log/suricata`.

    sudo apt-get install libyaml-dev
    wget https://www.openinfosecfoundation.org/download/suricata-4.0.3.tar.gz
    tar xzvf suricata-4.0.0.tar.gz
    cd suricata-4.0.0
    ./configure
    make
    make install

now, enter this command test that suricata is working correctly:
`ps aux| grep suricata`


<a id="orgb352335"></a>

### Snort

1.  Installing Snort for you system

    1.  Installing the Data Acquisition Library (DAQ)
        
            sudo apt-get install aptitude
            sudo aptitude install -y gcc flex bison make libpcap-dev libdnet-dev libdumbnet-dev libpcre3-dev libghc-zlib-dev
            wget https://www.snort.org/downloads/snort/daq-2.0.6.tar.gz
            tar -xvzf daq-2.0.6.tar.gz
            cd daq-2.0.6
            ./configure
            sudo make && sudo make install
    2.  Installing Snort
    
    Installing from source  <https://www.snort.org/downloads/>
    
        wget https://www.snort.org/downloads/snort/snort-2.9.11.tar.gz
        tar -xvzf snort-2.9.11.tar.gz
        cd snort-2.9.11/
        ./configure --enable-sourcefire && make && sudo make install
    
    1.  Once we have Snort installed, we need to make sure that our shared libraries are up to date. We can do this using the command:
    
    `sudo ldconfig`
    The resulting output will resemble the following:
    
        sudo snort --v
        
          ,,_     -*> Snort! <*-
         o"  )~   Version 2.9.11 GRE (Build 125)
          ''''    By Martin Roesch & The Snort Team: http://www.snort.org/contact#team
                  Copyright (C) 2014-2017 Cisco and/or its affiliates. All rights reserved.
                  Copyright (C) 1998-2013 Sourcefire, Inc., et al.
                  Using libpcap version 1.8.1
                  Using PCRE version: 8.39 2016-06-14
                  Using ZLIB version: 1.2.8
    
    1.  How to setup the configuration files
    2.  Snort on Debian gets installed to `/usr/local/bin/snort` directory. So we need to create a symlink:
        `sudo ln -s /usr/local/bin/snort /usr/sbin/snort`
    3.  To create a new user and group for running as root user
    
        sudo groupadd snort
        sudo useradd snort -r -s /sbin/nologin -c SNORT_IDS -g snort
    
    -   After we create the directories(Such as log files and config files) and the rules:
    
    before we can add any rules, we need a place to store the dynamic rules. 
    
    `sudo mkdir /usr/local/lib/snort_dynamicrules`
    
        sudo mkdir /etc/snort
        sudo mkdir /etc/snort/rules
        sudo mkdir /etc/snort/preproc_rules
        sudo touch /etc/snort/rules/white_list.rules /etc/snort/rules/black_list.rules /etc/snort/rules/local.rules
        
        sudo chmod -R 5775 /etc/snort
        sudo chmod -R 5775 /var/log/snort
        sudo chmod -R 5775 /usr/local/lib/snort_dynamicrules
        sudo chown -R snort:snort /etc/snort
        sudo chown -R snort:snort /var/log/snort
        sudo chown -R snort:snort /usr/local/lib/snort_dynamicrules
    
    Copy over the configuration files from `/home/User/snort-2.9.11/etc`
    
        sudo cp *.conf* /etc/snort
        sudo cp *.map /etc/snort

2.  Installing Snort3(++) for you system

    -   build-essential: provides the build tools (GCC and the like) to compile software.
    -   bison, flex: parsers required by DAQ (DAQ is installed later below).
    -   libpcap-dev: Library for network traffic capture required by Snort.
    -   libpcre3-dev: Library of functions to support regular expressions required by Snort.
    -   libdumbnet-dev: the libdnet library provides a simplified, portable interface to several low-level networking routines. Many guides for installing Snort install this library from source, although that is not necessary.
    -   zlib1g-dev: A compression library required by Snort.
    -   liblzma-dev: Provides decompression of swf files (adobe flash)
    -   openssl and libssl-dev: Provides SHA and MD5 file signatures
    -   1. Installing the Data Acquisition Library (DAQ)
        First we need to install all the Snort pre-requisites from the Debian repositories:
    
        sudo apt-get install -y build-essential autotools-dev libdumbnet-dev libluajit-5.1-dev libpcap-dev libpcre3-dev zlib1g-dev pkg-config libhwloc-dev cmake
        sudo apt-get install -y liblzma-dev openssl libssl-dev cpputest libsqlite3-dev
        sudo apt-get install -y bison flex
        sudo apt-get install -y libtool git autoconf
        sudo apt-get install -y asciidoc dblatex source-highlight
    
    create a directory to save the downloaded tarball 
    
    1.  DAQ
    
             mkdir ~/snort_src
             cd ~/snort_src
             wget https://www.snort.org/downloads/snortplus/daq-2.2.2.tar.gz
             tar -xvzf daq-2.2.2.tar.gz
             cd daq-2.2.2/
             ./configure
             make
             sudo make install
            
             cd ~/snort_src
             wget http://downloads.sourceforge.net/project/safeclib/libsafec-10052013.tar.gz
             tar -xzvf libsafec-10052013.tar.gz
             cd libsafec-10052013
             ./configure
             make
             sudo make install
            
             cd ~/snort_src
             wget http://www.colm.net/files/ragel/ragel-6.10.tar.gz
             tar -xzvf ragel-6.10.tar.gz
             cd ragel-6.10
             ./configure
             make
             sudo make install
            
             ;;;Download the Boost 1.64 libraries, but do not install:
            
             cd ~/snort_src
             wget https://dl.bintray.com/boostorg/release/1.64.0/source/boost_1_64_0.tar.gz
             tar -xvzf boost_1_64_0.tar.gz
            
            
             cd ~/snort_src
             wget https://github.com/01org/hyperscan/archive/v4.5.2.tar.gz
             tar -xvzf v4.5.2.tar.gz
            
             cd hyperscan-4.5.2
             cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DBOOST_ROOT=~/snort_src/boost_1_64_0/ ../hyperscan-4.5.2
            
             make
             sudo make install
            
            Run the following command to update shared libraries:
             sudo ldconfig
            
             git clone  https://github.com/snortadmin/snort3.git
             cd snort3/
             autoreconf -isvf 
             ./configure --prefix=/opt/snort
             make
             sudo make install
        
        create a symlink to /usr/sbin/snort:
        
        `sudo ln -s /opt/snort/bin/snort /usr/sbin/snort`
        
        ;; Set up the environment
        
            sh -c "echo 'export LUA_PATH=/opt/snort/include/snort/lua/\?.lua\;\;' >> ~/.bashrc"
            sh -c "echo 'export SNORT_LUA_PATH=/opt/snort/etc/snort' >> ~/.bashrc"
        
        To run Snort on Debian as root access
        
            sudo visudo
            Defaults env_keep += "LUA_PATH SNORT_LUA_PATH"
        
        The resulting output will resemble the following:
        
        `sudo snort -V`:
        
            
             ,,_     -*> Snort++ <*-
            o"  )~   Version 3.0.0 (Build 241) from 2.9.11
             ''''    By Martin Roesch & The Snort Team
                     http://snort.org/contact#team
                     Copyright (C) 2014-2017 Cisco and/or its affiliates. All rights reserved.
                     Copyright (C) 1998-2013 Sourcefire, Inc., et al.
                     Using DAQ version 2.2.2
                     Using LuaJIT version 2.0.4
                     Using OpenSSL 1.1.0f  25 May 2017
                     Using libpcap version 1.8.1
                     Using PCRE version 8.39 2016-06-14
                     Using ZLIB version 1.2.8
                     Using Hyperscan version 4.5.2 2017-12-30
                     Using LZMA version 5.2.2
    
    2.  Using snort3-community rules
    
            wget https://www.snort.org/downloads/community/snort3-community-rules.tar.gz
            sudo tar -xvf ~/snort3-community-rules.tar.gz -C ~/
            sudo cp ~/snort3-community-rules/* /etc/snort/rules
            sudo mkdir /opt/snort/etc/snort/rules                   
            sudo cp snort3-community.rules /opt/snort/etc/snort/rules/
            sudo cp sid-msg.map /opt/snort/etc/snort/rules/          
        
        -   Snort 2.9X we were stored the rule in `/etc/snort/` That's different with `/opt/snort/etc/snort/rules/`
        
        -   test that snort load these rules correctly
        
        `sudo snort -c snort.lua -R /opt/snort/etc/snort/rules/snort3-community.rules`
        
        -   if you got this issue like this:
        
        FATAL: can't init snort.lua: cannot open /snort<sub>defaults.lua</sub>: No such file or directory
        Fatal Error, Quitting。
        
        -   Now, Change you file name that snort can load rules.
        
        cd /opt/snort/etc/snort
        sudo mv snort<sub>defaults.lua</sub> ./snort<sub>defaults.lua</sub>
        sudo mv file<sub>magic.lua</sub> ./file<sub>magic.lua</sub>
        
        -   if you still have had this problem, check snort of the `file_magic config dir`.
        
        Then test again:
        `sudo snort -c snort.lua -R /opt/snort/etc/snort/rules/snort3-community.rules`
        
        -   The output information:
        
        Snort successfully validated the configuration (with 0 warnings).
        o")~   Snort exiting
    
    3.  Installing OpenAppID
    
        first of all, download the OpenAppID detector package:
        
            cd ~/snort_src/
            wget https://www.snort.org/downloads/openappid/6329
            tar -xzvf 6329
            sudo cp -R odp /opt/snort/lib/
        
            sudo nano /opt/snort/etc/snort/snort.lua
            ## find appid this line and add appid dir to here:
            
            appid =
            {
                app_detector_dir = '/opt/snort/lib'
            }
        
        Now, Test configuration:
        `sudo snort -c /opt/snort/etc/snort/snort.lua --warn-all`

3.  CSV Output

    First of all, Setting up the format csv file for snort, lets just copy this command into the configuration file stored in `/etc/snort/snort.config`
    
    -   Snort 2.9X:
    
    ~output alert<sub>csv</sub>: /var/log/snort/alert.csv timestamp,sig<sub>id,sig</sub><sub>rev,msg,proto,src,srcport,dst,dstport,ethsrc,ethdst,ethlen,tcpflags,ttl,iplen</sub>~r
    
    -   Snort3:
        see the available configuration like this:
        `$ snort --help-config alert_csv`
    
    then:
    `snort -c /etc/snort/config/snort.lua -i ens192 -l /var/log/snort -A --lua "alert_csv = { fields = ’pkt_num gid sid rev’, separator = ’\t’ }"`
    
        • bool alert_csv.file = false: output to alert_csv.txt instead of stdout
        • multi alert_csv.fields = timestamp pkt_num proto pkt_gen dgm_len dir src_ap dst_ap rule action: selected fields will be output
        in given order left to right { action | dir | dgm_len | dst_addr | dst_ap | dst_port | eth_dst | eth_len | eth_src | eth_type | gid |
        icmp_code | icmp_id | icmp_seq | icmp_type | iface | ip_id | ip_len | msg | pkt_gen | pkt_num | proto | rev | rule | sid | src_addr |
        src_ap | src_port | tcp_ack | tcp_flags | tcp_len | tcp_seq | tcp_win | timestamp | tos | ttl | udp_len }
        • int alert_csv.limit = 0: set maximum size in MB before rollover (0 is unlimited) { 0: }
    
    For more information about： 
    [Snort output format](http://manual-snort-org.s3-website-us-east-1.amazonaws.com/node21.html#SECTION00366100000000000000) **Click this url**
    
    \##Snort 2.9X  running bro
    `sudo systemctl restart snort.service`
    `sudo snort -i ens192 -c /etc/snort/snort.conf`
    
    The **-i** option you can enter the interface card whatever you like to use to sniff packets form. and the **-c** point which configure file used.
    
    \###Snort 2.9x
    before type the following command to restart the Snort:
    `sudo systemctl restart snort.serivce`
    Then, checking test.
    `ps aux | grep snort`


<a id="org085730d"></a>

# Adding Patterns for ELk


<a id="org3f78268"></a>

## Bro

1.  [Logstrash Bro config file](#org1087f18)
2.  ensure that you are finishing the bro config for logstrash (1.)
3.  Index Patterns

cleaning console buffer that you are make sure able to load bro conf correctly.

![img](img/Console%20-%20Kibana%20.png)

finally,

![img](img/index%20bro.png)


<a id="orgcd09d14"></a>

### More information


<a id="org4322c5e"></a>

## Suricata & Snort

-   if you did install suricata with debian. click this link for you help. <http://ral-arturo.org/2017/08/19/suricata-debian.html>

-   Checking Suricata status same like this:

`sudo systemctl status suricata`

● suricata.service - Suricata IDS/IDP daemon
Loaded: loaded (/lib/systemd/system/suricata.service; disabled; vendor preset: enabled)
Active: inactive (dead)
Docs: man:suricata(8)
man:suricatasc(8)

<https://redmine.openinfosecfoundation.org/projects/suricata/wiki>

— go to Syslog-ng conf file, add following it here in `sensor host`:

    
    source s_ids_suri { file("/var/log/suricata/eve.json" flags(no-parse) program_override("suricata")); };
    source s_ids_snort { file("/var/log/snort/alert.csv" flags(no-parse) program_override("snort")); };
    
    destination d_ids { network("192.168.1.8" port(1001)); }; #manager host
    
    log {
            source(s_ids_snort);
            source(s_ids_suri);
            log { destination(d_ids); };
    };

Then restart service:
`sudo systemctl restart syslog-ng.service`
and Adding partner same like this way to check:

![img](img/index%20bro.png)


<a id="org653ee21"></a>

### More Information

For more information about Suricata, please see:


<a id="orgddc8f71"></a>

# Adding a dashboard of data type


<a id="org9e3bb50"></a>

## Bro

-   first of all, we need to create some visualizations.

![img](img/Kibana%202017-12-24%2001-46-08.png)

-   Pie of Bro file types

![img](img/Bro%20File%20Types.png)

-   Line of Bro events over time

![img](img/Bro%20Events%20over%20time.png)

-   Metic of bro log

![img](img/Metric.png)

-   Visual Builder of Src Bytes / Dst Bytes
    1.  personally, Clone series from src to dst and changed  color to pink

![img](img/Src%20Bytes%20-%20Kibana%202017-12-25%2002-06-02.png)

1.  Second of all,changed bar in option

![img](img/src%20bar.png)

1.  finally, organizing you

