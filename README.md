# Active-Active-High-Availability-Cluster-Server

Active Active High Availability CLuster Server applying Load Balancer and High Availability to Server

<h3>Step:<br><br>
1. Provide 4 mini-pc Raspberry Pi<br>
2. Configure Raspi 1 and Raspi 2 as The Master Node.<br>
3. Configure Raspi 3 and Raspi 4 as The Slave Node.<br>
4. In Slave Node Install Apache2 </h3>
<br>We used Linux Ubuntu as the OS of the Raspberry Pi,
So the command line is: 

```
sudo apt install apache2
```
 
<h3>5. In Slave Node do config at apache2.conf</h3>
After you install the apache2 software, do
<br>

```
sudo vim /etc/apache2/apache2.conf
```

<br>
after open apache2.conf, insert this statement:
<br>

```
#LogFormat "%h %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined 
LogFormat "%{X-Forwarded-For}i %l %u %t \"%r\" %>s %b \"%{Referer}i\" \"%{User-Agent}i\"" combined
```

<h3>6. In Slave Node do config at default </h3>
You can find default.conf at 

```
sudo vim /etc/apache2/sites-available/000-default.conf
```

after you open the default.conf, insert this statement:

```
SetEnvIf Request_URI "^/check\.txt$" dontlog
CustomLog /var/log/apache2/access.log combined env=!dontlog
```

<h3>7. Restart the configuration </h3>

```
sudo systemctl restart apache2
#check_status
sudo systemctl status apache2
```
<h3>8. create the file check.txt </h3>

```
sudo touch /var/www/check.txt
```

<h3>9. Then in Master Node, at Raspi node 1 and Raspi node 2 install haproxy</h3>

```
sudo apt install haproxy
```

<h3>10. edit haproxy.cfg at </h3>

```
sudo vim /etc/haproxy/haproxy.cfg
```

Copy paste command at haproxy.cfg in this repository to your .cfg

<h3> 11. edit haproxy's default </h3>
 
 ```
 sudo vim /etc/default/haproxy
 ```
 
insert this code:
 
 ```
 ENABLED=1
 ```
 
<h3>12. Restart the configuration </h3>

 ```
 sudo systemctl restart haproxy
 #check_status
 sudo systemctl status haproxy
 ```

<h3>12. Install keepalived </h3>
 
 ```
 sudo apt install getalived
 ```
 
<h3>13. insert keepalived.conf </h3>

open at 
 ```
 sudo vim /etc/keepalived/keepalived.conf
 ```
add this code:
  
 ```
 vrrp_script chk_haproxy {
 script "killall -0 haproxy" # check the haproxy process
 interval 2 # every 2 seconds
 weight 2 # add 2 points if OK
}

 vrrp_instance VI_1 {
  interface eth0 # interface to monitor
  state MASTER # MASTER on haproxy1, BACKUP on haproxy2
  virtual_router_id 51
  priority 101 # 101 on haproxy1, 100 on haproxy2
   virtual_ipaddress {
    192.168.0.100 # virtual ip address 
   }
  track_script {
    chk_haproxy
  }
}
```

<h3>14. Restart the configuration </h3>

```
sudo systemctl restart keepalived
#check_status
sudo systemctl status keepalived
```
<h3>15. Test the floating IP </h3>

```
ip a
```

<h3>16. Test your server </h3>

open your ip 192.168.100.35 at browser
