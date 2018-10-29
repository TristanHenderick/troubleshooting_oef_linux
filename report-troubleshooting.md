# Enterprise Linux Lab Report - Troubleshooting

- Student name: Tristan Henderick
- Class/group: TIN-TI-3B (Gent), TIN2-TI-3B (Aalst), TIN-TILE (Afstandsleren) [weglaten wat niet past]

## Instructions

- Write a detailed report in the "Report" section below (in Dutch or English)
- Use correct Markdown! Use fenced code blocks for commands and their output, terminal transcripts, ...

- The different phases in the bottom-up troubleshooting process are described in their own subsections (heading of level 3, i.e. starting with `###`) with the name of the phase as title.

- Every step is described in detail:

    - describe what is being tested
    - give the command, including options and arguments, needed to execute the test, or the absolute path to the configuration file to be verified
    - give the expected output of the command or content of the configuration file (only the relevant parts are sufficient!)
    - if the actual output is different from the one expected, explain the cause and describe how you fixed this by giving the exact commands or necessary changes to configuration files

- In the section "End result", describe the final state of the service:
    - copy/paste a transcript of running the acceptance tests
    - describe the result of accessing the service from the host system
    - describe any error messages that still remain

## Report

### Phase 1: Network Access layer

-check virtual network adapter type & settings:
    
    adapter 1 attached to NAT.
    adapter 2 attached to Host-only adapter #2. 

- command `ip link` to check state of NICs

        enp0s8 is down
        
        => check virtual box if cable connected, cable was not connected. connected cable
        
        enp0s8 is now up

### Phase 2: Internet Layer

-command `ip a` to list ip addresses

    enp0s8 has no ipv4 address
    
    `cat /etc/sysconfig/network-scripts/ifcfg-enp0s8`
    
    ifcfg-enp0s8 has IPADRR instead of IPADDR => fix spelling error
    
    `service network restart`
    
    `ip a`
    
    enp0s8 now has the correct ipv4 address
    
- command `ip route` to show default gateway

        default via 10.0.2.2
        seems fine
        
- command `getent ahosts www.google.com` to test DNS
        
        172.217.17.68 STREAM www.google.com
        
        seems fine
        
        
### Phase 3: transport layer


-check if services are running

    `sudo service nginx status`
    =>service is not running
    `sudo systemctl start ngingx.service`
    => failed; see "systemctl status nginx.service" and "journalctl -xe" for details
    `sudo systemctl status nginx.service`
    => failed to start the nginx http and reverse proxy server
    
    `sudo vi /etc/nginx/nginx.conf`
    
    => server listen 8443 moet 443 zijn. ook een typfout in ssl_certificate pad
    
    => service is running
    
-firewall settings

    
    see if port 443 is open
    `sudo firewall-cmd --list-all`
    => it is not
    
    `sudo firewall-cmd --add-service=https --permanent`
    
    `sudo firewall-cmd --add-port=443/tcp --permanent`
    
    `sudo systemctl restart firewalld`
    
    check again
    ` sudo firewall-cmd --list-all`
    port 443 is now open
    

### Phase 4: application layer

-check the log files

    `sudo journalctl -f -u nginx.service`
    => failed to read PID from file /run/nginx.pid:Invalid argument.
    this error was caused by the virtual machine being connected to the wrong adapter(host-only#2). connecting it to the right adapter (host-only adapter) and    rebooting the system fixed this error
    

-validate the settings of the configuration file 

    `sudo /usr/sbin/nginx -t -c /etc/nginx/nginx.conf`
    
    => syntax is ok; test is successful

...

## End result

can connect to website https://192.168.56.42/

## Resources

List all sources of useful information that you encountered while completing this assignment: books, manuals, HOWTO's, blog posts, etc.

https://bertvv.github.io/linux-network-troubleshooting/

https://bertvv.github.io/linux-network-troubleshooting/tldr-checklist.html

https://hogenttin.github.io/elnx-syllabus/troubleshooting/#/title-slide
