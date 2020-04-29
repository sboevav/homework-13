#  SELinux - когда все запрещено 

## 

[vagrant@otuslinux ~]$ sudo systemctl start nginx
[vagrant@otuslinux ~]$ systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2020-04-29 17:41:35 UTC; 3s ago
  Process: 4859 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 4856 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 4855 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 4861 (nginx)
   CGroup: /system.slice/nginx.service
           ├─4861 nginx: master process /usr/sbin/nginx
           └─4862 nginx: worker process


[vagrant@otuslinux ~]$ curl 192.168.11.101
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
  <title>Welcome to CentOS</title>
  <style rel="stylesheet" type="text/css"> 
...
  </style>

</head>

<body>
...
</body>
</html>




[vagrant@otuslinux ~]$ sudo systemctl stop nginx



[vagrant@otuslinux ~]$ sudo vi /etc/nginx/nginx.conf
[root@otuslinux vagrant]# cat /etc/nginx/nginx.conf
...

    server {
        listen       8182 default_server;
...



[root@otuslinux vagrant]# systemctl start nginx
Job for nginx.service failed because the control process exited with error code. See "systemctl status nginx.service" and "journalctl -xe" for details.
[root@otuslinux vagrant]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: failed (Result: exit-code) since Wed 2020-04-29 18:53:48 UTC; 8s ago
  Process: 5057 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
  Process: 5056 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)

Apr 29 18:53:48 otuslinux systemd[1]: Starting The nginx HTTP and reverse p.....
Apr 29 18:53:48 otuslinux nginx[5057]: nginx: the configuration file /etc/n...ok
Apr 29 18:53:48 otuslinux nginx[5057]: nginx: [emerg] bind() to 0.0.0.0:818...d)
Apr 29 18:53:48 otuslinux nginx[5057]: nginx: configuration file /etc/nginx...ed
Apr 29 18:53:48 otuslinux systemd[1]: nginx.service: control process exited...=1
Apr 29 18:53:48 otuslinux systemd[1]: Failed to start The nginx HTTP and re...r.
Apr 29 18:53:48 otuslinux systemd[1]: Unit nginx.service entered failed state.
Apr 29 18:53:48 otuslinux systemd[1]: nginx.service failed.
Hint: Some lines were ellipsized, use -l to show in full.




[root@otuslinux vagrant]# audit2why < /var/log/audit/audit.log
...
type=AVC msg=audit(1588186428.139:943): avc:  denied  { name_bind } for  pid=5057 comm="nginx" src=8182 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0

	Was caused by:
	The boolean nis_enabled was set incorrectly. 
	Description:
	Allow nis to enabled

	Allow access by executing:
	# setsebool -P nis_enabled 1



Вариант 1
[root@otuslinux vagrant]# semanage port -l | grep 80
amanda_port_t                  tcp      10080-10083
amanda_port_t                  udp      10080-10082
cyphesis_port_t                tcp      6767, 6769, 6780-6799
geneve_port_t                  tcp      6080
hadoop_namenode_port_t         tcp      8020
hplip_port_t                   tcp      1782, 2207, 2208, 8290, 8292, 9100, 9101, 9102, 9220, 9221, 9222, 9280, 9281, 9282, 9290, 9291, 50000, 50002
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
jabber_interserver_port_t      tcp      5269, 5280
jboss_management_port_t        tcp      4447, 4712, 7600, 9123, 9990, 9999, 18001
luci_port_t                    tcp      8084
mongod_port_t                  tcp      27017-27019, 28017-28019
mxi_port_t                     tcp      8005
mxi_port_t                     udp      8005
oa_system_port_t               tcp      8022
oa_system_port_t               udp      8022
ocsp_port_t                    tcp      9080
pki_ca_port_t                  tcp      829, 9180, 9701, 9443-9447
pki_kra_port_t                 tcp      10180, 10701, 10443-10446
pki_ocsp_port_t                tcp      11180, 11701, 11443-11446
pki_tks_port_t                 tcp      13180, 13701, 13443-13446
preupgrade_port_t              tcp      8099
prosody_port_t                 tcp      5280-5281
soundd_port_t                  tcp      8000, 9433, 16001
speech_port_t                  tcp      8036
transproxy_port_t              tcp      8081
us_cli_port_t                  tcp      8082, 8083
us_cli_port_t                  udp      8082, 8083
xen_port_t                     tcp      8002
zope_port_t                    tcp      8021




[root@otuslinux vagrant]# semanage port -a -t http_port_t -p tcp 8182

[root@otuslinux vagrant]# systemctl start nginx
[root@otuslinux vagrant]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2020-04-29 18:58:00 UTC; 8s ago
  Process: 5080 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 5078 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 5077 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 5082 (nginx)
   CGroup: /system.slice/nginx.service
           ├─5082 nginx: master process /usr/sbin/nginx
           └─5083 nginx: worker process

Apr 29 18:58:00 otuslinux systemd[1]: Starting The nginx HTTP and reverse p.....
Apr 29 18:58:00 otuslinux nginx[5078]: nginx: the configuration file /etc/n...ok
Apr 29 18:58:00 otuslinux nginx[5078]: nginx: configuration file /etc/nginx...ul
Apr 29 18:58:00 otuslinux systemd[1]: Failed to read PID from file /run/ngi...nt
Apr 29 18:58:00 otuslinux systemd[1]: Started The nginx HTTP and reverse pr...r.
Hint: Some lines were ellipsized, use -l to show in full.



[root@otuslinux vagrant]# curl 192.168.11.101:8182
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
  <title>Welcome to CentOS</title>
  <style rel="stylesheet" type="text/css"> 
...
  </style>

</head>

<body>
...
</body>
</html>




[root@otuslinux vagrant]# systemctl stop nginx
[root@otuslinux vagrant]# semanage port -d -t http_port_t -p tcp 8182
[root@otuslinux vagrant]# semanage port -l | grep http_port_t
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988



Вариант 2
[root@otuslinux vagrant]# setsebool -P nis_enabled 1
[root@otuslinux vagrant]# systemctl start nginx
[root@otuslinux vagrant]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2020-04-29 19:21:08 UTC; 9s ago
  Process: 5164 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 5161 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 5160 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 5166 (nginx)
   CGroup: /system.slice/nginx.service
           ├─5166 nginx: master process /usr/sbin/nginx
           └─5167 nginx: worker process

Apr 29 19:21:08 otuslinux systemd[1]: Starting The nginx HTTP and reverse p.....
Apr 29 19:21:08 otuslinux nginx[5161]: nginx: the configuration file /etc/n...ok
Apr 29 19:21:08 otuslinux nginx[5161]: nginx: configuration file /etc/nginx...ul
Apr 29 19:21:08 otuslinux systemd[1]: Failed to read PID from file /run/ngi...nt
Apr 29 19:21:08 otuslinux systemd[1]: Started The nginx HTTP and reverse pr...r.
Hint: Some lines were ellipsized, use -l to show in full.

[root@otuslinux vagrant]# curl 192.168.11.101:8182
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
  <title>Welcome to CentOS</title>
  <style rel="stylesheet" type="text/css"> 
...
  </style>

</head>

<body>
...
</body>
</html>

[root@otuslinux vagrant]# setsebool -P nis_enabled 0
[root@otuslinux vagrant]# systemctl stop nginx




Вариант 3

[root@otuslinux vagrant]# sealert -a /var/log/audit/audit.log

(setroubleshoot:5497): Gtk-WARNING **: 19:35:47.131: Locale not supported by C library.
	Using the fallback 'C' locale.
100% done
found 3 alerts in /var/log/audit/audit.log
--------------------------------------------------------------------------------
...

SELinux is preventing /usr/sbin/nginx from name_bind access on the tcp_socket port 8182.

*****  Plugin bind_ports (92.2 confidence) suggests   ************************

If you want to allow /usr/sbin/nginx to bind to network port 8182
Then you need to modify the port type.
Do
# semanage port -a -t PORT_TYPE -p tcp 8182
    where PORT_TYPE is one of the following: http_cache_port_t, http_port_t, jboss_management_port_t, jboss_messaging_port_t, ntop_port_t, puppet_port_t.

*****  Plugin catchall_boolean (7.83 confidence) suggests   ******************

If you want to allow nis to enabled
Then you must tell SELinux about this by enabling the 'nis_enabled' boolean.

Do
setsebool -P nis_enabled 1

*****  Plugin catchall (1.41 confidence) suggests   **************************

If you believe that nginx should be allowed name_bind access on the port 8182 tcp_socket by default.
Then you should report this as a bug.
You can generate a local policy module to allow this access.
Do
allow this access for now by executing:
# ausearch -c 'nginx' --raw | audit2allow -M my-nginx
# semodule -i my-nginx.pp


Additional Information:
Source Context                system_u:system_r:httpd_t:s0
Target Context                system_u:object_r:unreserved_port_t:s0
Target Objects                port 8182 [ tcp_socket ]
Source                        nginx
Source Path                   /usr/sbin/nginx
Port                          8182
Host                          <Unknown>
Source RPM Packages           nginx-1.16.1-1.el7.x86_64
Target RPM Packages           
Policy RPM                    selinux-policy-3.13.1-229.el7_6.12.noarch
Selinux Enabled               True
Policy Type                   targeted
Enforcing Mode                Enforcing
Host Name                     otuslinux
Platform                      Linux otuslinux 3.10.0-957.12.2.el7.x86_64 #1 SMP
                              Tue May 14 21:24:32 UTC 2019 x86_64 x86_64
Alert Count                   3
First Seen                    2020-04-29 18:53:48 UTC
Last Seen                     2020-04-29 19:23:33 UTC
Local ID                      4aaad2d0-cafc-4081-af70-c63b4fd43a7f

Raw Audit Messages
type=AVC msg=audit(1588188213.338:978): avc:  denied  { name_bind } for  pid=5194 comm="nginx" src=8182 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0


type=SYSCALL msg=audit(1588188213.338:978): arch=x86_64 syscall=bind success=no exit=EACCES a0=6 a1=5642611456a8 a2=10 a3=7fff31a20570 items=0 ppid=1 pid=5194 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm=nginx exe=/usr/sbin/nginx subj=system_u:system_r:httpd_t:s0 key=(null)

Hash: nginx,httpd_t,unreserved_port_t,tcp_socket,name_bind





[root@otuslinux vagrant]# ausearch -c 'nginx' --raw | audit2allow -M my-nginx
******************** IMPORTANT ***********************
To make this policy package active, execute:

semodule -i my-nginx.pp

[root@otuslinux vagrant]# semodule -i my-nginx.pp



[root@otuslinux vagrant]# systemctl start nginx
[root@otuslinux vagrant]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
   Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; vendor preset: disabled)
   Active: active (running) since Wed 2020-04-29 19:40:41 UTC; 9s ago
  Process: 5573 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
  Process: 5571 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
  Process: 5570 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
 Main PID: 5575 (nginx)
   CGroup: /system.slice/nginx.service
           ├─5575 nginx: master process /usr/sbin/nginx
           └─5576 nginx: worker process

Apr 29 19:40:41 otuslinux systemd[1]: Starting The nginx HTTP and reverse p.....
Apr 29 19:40:41 otuslinux nginx[5571]: nginx: the configuration file /etc/n...ok
Apr 29 19:40:41 otuslinux nginx[5571]: nginx: configuration file /etc/nginx...ul
Apr 29 19:40:41 otuslinux systemd[1]: Failed to parse PID from file /run/ng...nt
Apr 29 19:40:41 otuslinux systemd[1]: Started The nginx HTTP and reverse pr...r.
Hint: Some lines were ellipsized, use -l to show in full.




[root@otuslinux vagrant]# curl 192.168.11.101:8182
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
<head>
  <title>Welcome to CentOS</title>
  <style rel="stylesheet" type="text/css"> 
...
  </style>

</head>

<body>
...
</body>
</html>

