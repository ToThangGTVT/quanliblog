
1. Dừng app đang chạy trên cổng 8080
``` shell
sudo kill -9 `sudo lsof -t -i:8080`
```
2. Mở thu mục config nginx Ubuntu
``` shell
cd /etc/nginx/sites-available
```
3. Restart nginx Ubuntu
``` shell
sudo systemctl restart nginx
```
4. Đọc log
``` shell
tail -f nohup.out
```
5. Đọc log service oci
``` shell
journalctl -u oci -f
```
6. Mở toàn bộ port ubuntu iptables
``` shell
sudo iptables -I INPUT -j ACCEPT
```
7. Tạo soft link nginx
``` shell
sudo ln -s /etc/nginx/sites-available/example.conf /etc/nginx/sites-enabled/
```
8. Cấu hình run java on service
``` shell
cd /etc/systemd/system
tạo file <nameservice>.service
================================
[Unit]
Description=oci_app
After=syslog.target
[Service]
WorkingDirectory=/usr/oci
ExecStart=java -jar oci-0.0.2.jar
SuccessExitStatus=143
Type=simple
ExecStop=/bin/kill -15 $MAINPID
[Install]
WantedBy=multi-user
```
9. Run/Stop service
``` shell
sudo systemctl start oci
sudo systemctl stop oci
sudo systemctl status oci
```
10. Multi app nginx
``` shell
server {
		listen ...;
		...
		location / {
			proxy_pass http://127.0.0.1:8080;
		}
		location /blog {
			rewrite ^/blog(.) /$1 break;
			proxy_pass http://127.0.0.1:8181;
		}
		location /mail {
			rewrite ^/mail(.) /$1 break;
			proxy_pass http://127.0.0.1:8282;
		}
}
```
11. Đổi tên file 
``` shell
mv [old_file_name] [new_file_name]
```
