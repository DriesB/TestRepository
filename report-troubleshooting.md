# Enterprise Linux Lab Report - Troubleshooting

- Student name: Dries Boone
- Class/group: TIN-TI-3B (Gent)

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

### Phase 1: Link Layer

Check if the 'cables' are connected

|Expected|Reality|
|:--|--:|
|All cables connected|Host-only adapter was not connected|

### Phase 2: Network Layer

Check the ip addresses with `ip a`

|Expected|Reality|
|:--|--:|
|ip enp0s3: `10.0.2.15/24` | `10.0.2.15/24` older version of VirtualBox expect `10.0.2.15/8` |
|ip enp0s8: `192.168.56.42/24`| `192.168.56.42/24` |

Check the default gateway with `ip r`

|Expected|Reality|
|:--|--:|
|`default via 10.0.2.2`| `default via 10.0.2.2` |
|ping respons from `10.0.2.2`| ping succesful|

Check the DNS with `cat /etc/resolv.conf`

|Expected|Reality|
|:--|--:|
|`nameserver 10.0.2.3`|`nameserver 10.0.2.3`|
|`search internal`|`search hogent.be`|
`sudo vi /etc/resolv.conf`
press 'i' to change the file
overwrite `hogent.be` with `internal` 
press escape
write `:wq`
Download the `dig utility` with `sudo yum install bind-utils`
Test the dns server with `dig www.google.be` -> works

### Phase 3: Transport Layers

Check the status of the service with `systemctl status nginx.service`

|Expected|Reality|
|:--|--:|
|`Active: running`|`Active: inactive (dead)`|

Start the service with `systemctl start nginx.service` -> failed
Something could be wrong with the config file, test with `sudo nginx -t` 
Error output: `nginx: [emerg] BIO_new_file ("/etc/tls/certs/nigxn.pem")... failed... error:02001002 .... error:2006d080:BIO routines: BIO_new_file:no such file`
View the config file with `sudo vi /etc/nginx/nginx.conf`

|Expected|Reality|
|:--|--:|
|"/etc/pki/tls/private/nginx.key"|"/etc/pki/tls/private/nginx.key"|
|"/etc/pki/tls/certs/nginx.pem"|"/etc/pki/tls/certs/nigxn.pem"|

Adjust the file accordingly
Start the service with `systemctl start nginx.service`
Check the service with `systemctl status nginx.service`

Check the (open) ports with `sudo ss -tlnp`

|Expected|Reality|
|:--|--:|
|port 80 listening|port 80 is listening|

Check the firewall with `sudo firewall-cmd --list-all`

|Expected|Reality|
|:--|--:|
|`services: http https`|`services: http`|
|`ports: 80/tcp` 8080/tcp|`ports: `|

Add the `https` service with `sudo firewall-cmd --add-service=https --permanent`
Add the 80 port with `sudo firewall-cmd --add-port=80/tcp --permanent`
Add the 8080 port with `sudo firewall-cmd --add-port=8080/tcp --permanent`
Restart the firewall with `sudo systemctl restart firewalld`

Check if php is installed with `which php` -> no php installed
`sudo yum install php php-fpm`


### Phase 4: Application Layer

Install a text based brower (lynx) with `sudo yum -y install lynx`
Use `lynx 127.0.0.1` (if everything is correct, this page should load correctly) -> Access denied

Use `sudo journalctl -b -f -u nginx.service` to see the logs, "Failed to read PID from file /run/nginx.pid: Invalid argument" service runs however

Nginx can run html but not php (´lynx 127.0.0.1/index.html´ works, ´lynx 127.0.0.1/index.php` doesn't)

Changed the php.ini `cgi.fix_pathinfo=0` to `cgi.fix_pathinfo=1` 
Restarted nginx -> didn't solve


## End result

- copy/paste a transcript of running the acceptance tests

4 tests, 1 failure
Apache should not be installed (from function `ensure_not_installed` in file 01-whitebox.bats, line 14, in test file 01-whitebox.bat, line 39) `ensure_not_installed httpd` failed

- describe the result of accessing the service from the host system

Time-out for both http and https

- describe any error messages that still remain

Access denied when trying to access a php page

## Resources

List all sources of useful information that you encountered while completing this assignment: books, manuals, HOWTO's, blog posts, etc.

https://github.com/HoGentTIN/elnx-cd-DriesB/blob/master/report/cheat-sheet.md
https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-centos-7
http://serverfault.com/questions/617616/nginx-fails-to-load-a-file-ssl-certificate-even-if-its-clearly-there/617632
https://bugs.launchpad.net/ubuntu/+source/nginx/+bug/1581864
https://askubuntu.com/questions/164627/nginx-php-fpm-access-denied-error/164702