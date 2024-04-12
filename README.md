# Cài đặt WAF cho HAProxy với ModSecurity
## 1. Tổng quan
- WAF là viết tắt của “Web Application Firewall”. Đây là một loại tường lửa (firewall) được sử dụng để
bảo vệ các ứng dụng web khỏi các cuộc tấn công bằng cách giám sát, chặn và phát hiện các hoạt
động bất thường. WAF có thể giúp ngăn chặn các cuộc tấn công như SQL injection, cross-site
scripting, và các cuộc tấn công khác nhằm vào các lỗ hổng bảo mật của ứng dụng web. Nó hoạt
động bằng cách phân tích các request HTTP và sử dụng các quy tắc (rule) để xác định các hành vi
độc hại.
- WAF là một phần quan trọng trong việc bảo vệ ứng dụng web và đảm bảo an toàn cho dữ liệu của
người dùng.
## 2. Yêu cầu
|Application   |Version     |
|:------------:|:--------:  |
|OS            |Ubuntu 20.04|
|HAProxy       |v2.7.0      |
|ModSecurity   |v3.0.9      |
### Cài đặt package dependencies
- Cài đặt một số dependencies phục vụ cho quá trình build các packages:
```
$ sudo apt-get -y install build-essential doxygen valgrind libyajl-dev libgeoip-dev liblmdb-dev liblua5.3-dev libpcre2-dev libxml2-dev libcurl4-openssl-dev libfuzzy-dev libevent-dev libpcre3-dev libssl-dev libsystemd-dev jq
```
## 3. Cài đặt
### Cài đặt ModSecurity
```
$ sudo mkdir /etc/modsecurity
$ wget https://github.com/SpiderLabs/ModSecurity/releases/download/v3.0.9/modsecurity-v3.0.9.tar.gz
$ tar xzvf modsecurity-v3.0.9.tar.gz
$ cd modsecurity-v3.0.9
$ ./configure --prefix=/usr/local/modsecurity-3.0.9 --with-lua --with-pcre2 --with-lmdb
$ make -j $(nproc)
$ sudo make install
$ sudo ln -s /usr/local/modsecurity-3.0.9 /usr/local/modsecurity
```
## Cài đặt spoa-modsecurity
```
$ git clone https://github.com/haproxy/spoa-modsecurity.git
$ cp -r /usr/local/modsecurity/include/modsecurity spoa-modsecurity/include
$ cd spoa-modsecurity
$ sed -i 's/modsecurity-2.9.1\/INSTALL\/include/\/usr\/local\/modsecurity\/include\/modsecurity/g' Makefile
$ sed -i 's/modsecurity-2.9.1\/INSTALL\/lib/\/usr\/local\/modsecurity\/lib/g' Makefile
$ make
$ sudo make install
```
- Tạo file modsec systemd
```
cat > /usr/lib/systemd/system/modsecurity.service <<EOF
[Unit]
Description=Modsecurity Standalone
After=network.target
 
[Service]
Environment=LD_LIBRARY_PATH=/usr/local/modsecurity/lib/
Environment="CONFIG=/etc/modsecurity/modsecurity.conf" "EXTRAOPTS=-d -n 1"
ExecStart=/usr/local/bin/modsecurity $EXTRAOPTS -f $CONFIG
ExecReload=/usr/local/bin/modsecurity $EXTRAOPTS -f $CONFIG
ExecReload=/bin/kill -USR2 $MAINPID
Restart=always
Type=simple
 
[Install]
WantedBy=multi-user.target
EOF
```
## Cài đặt OWASP ModSecurity CRS
```
$ wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v3.3.4.tar.gz -O coreruleset-v3.3.4.tar.gz
$ tar xzvf coreruleset-v3.3.4.tar.gz
$ sudo mv coreruleset-3.3.4 /usr/local/coreruleset-3.3.4
$ sudo ln -sf /usr/local/coreruleset-3.3.4 /usr/local/coreruleset
$ cd /usr/local/coreruleset
$ sudo mv crs-setup.conf.example crs-setup.conf
$ sudo mv REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
$ sudo mv RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
```
- Thêm các dòng sau vào file /etc/modsecurity/modsecurity.conf
```
cat >> /etc/modsecurity/modsecurity.conf <<EOF
include /usr/local/coreruleset/crs-setup.conf
include /usr/local/coreruleset/rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
include /usr/local/coreruleset/rules/REQUEST-901-INITIALIZATION.conf
include /usr/local/coreruleset/rules/REQUEST-905-COMMON-EXCEPTIONS.conf
include /usr/local/coreruleset/rules/REQUEST-910-IP-REPUTATION.conf
include /usr/local/coreruleset/rules/REQUEST-911-METHOD-ENFORCEMENT.conf
include /usr/local/coreruleset/rules/REQUEST-912-DOS-PROTECTION.conf
include /usr/local/coreruleset/rules/REQUEST-913-SCANNER-DETECTION.conf
include /usr/local/coreruleset/rules/REQUEST-920-PROTOCOL-ENFORCEMENT.conf
include /usr/local/coreruleset/rules/REQUEST-921-PROTOCOL-ATTACK.conf
include /usr/local/coreruleset/rules/REQUEST-930-APPLICATION-ATTACK-LFI.conf
include /usr/local/coreruleset/rules/REQUEST-931-APPLICATION-ATTACK-RFI.conf
include /usr/local/coreruleset/rules/REQUEST-932-APPLICATION-ATTACK-RCE.conf
include /usr/local/coreruleset/rules/REQUEST-933-APPLICATION-ATTACK-PHP.conf
include /usr/local/coreruleset/rules/REQUEST-941-APPLICATION-ATTACK-XSS.conf
include /usr/local/coreruleset/rules/REQUEST-942-APPLICATION-ATTACK-SQLI.conf
include /usr/local/coreruleset/rules/REQUEST-943-APPLICATION-ATTACK-SESSION-FIXATION.conf
include /usr/local/coreruleset/rules/REQUEST-949-BLOCKING-EVALUATION.conf
include /usr/local/coreruleset/rules/RESPONSE-950-DATA-LEAKAGES.conf
include /usr/local/coreruleset/rules/RESPONSE-951-DATA-LEAKAGES-SQL.conf
include /usr/local/coreruleset/rules/RESPONSE-952-DATA-LEAKAGES-JAVA.conf
include /usr/local/coreruleset/rules/RESPONSE-953-DATA-LEAKAGES-PHP.conf
include /usr/local/coreruleset/rules/RESPONSE-954-DATA-LEAKAGES-IIS.conf
include /usr/local/coreruleset/rules/RESPONSE-959-BLOCKING-EVALUATION.conf
include /usr/local/coreruleset/rules/RESPONSE-980-CORRELATION.conf
include /usr/local/coreruleset/rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf
EOF
```
## Cài đặt HAProxy
```
$ wget https://github.com/haproxy/haproxy/archive/refs/tags/v2.7.0.tar.gz -O haproxy-v2.7.0.tar.gz
$ tar xzvf haproxy-v2.7.0.tar.gz
$ cd haproxy-2.7.0
$ make -j $(nproc) TARGET=linux-glibc USE_OPENSSL=1 USE_LUA=1 USE_PCRE=1 USE_SYSTEMD=1
$ sudo make install
```
- Tạo file configuration cho modsecurity
```
cat > /etc/haproxy/spoe-modsecurity.conf <<EOF
[modsecurity]
    spoe-agent modsecurity-agent
        messages check-request
        option var-prefix modsec
        timeout hello 100ms
        timeout idle 30s
        timeout processing 15ms
        use-backend spoe-modsecurity

    spoe-message check-request
        args unique-id src src_port dst dst_port method path query req.ver req.hdrs_bin req.body_size req.body
        event on-frontend-http-request
EOF
```
- Cập nhật file /etc/haproxy/haproxy.cfg
```
$ vim /etc/haproxy/haproxy.cfg
 
frontend https-in
    unique-id-format % { +X } o\ %ci:%cp_%fi:%fp_%Ts_%rt:%pid
    unique-id-header X-Unique-ID
    log-format "%ci:%cp [%tr] %ft %b/%s %TR/%Tw/%Tc/%Tr/%Ta %ST %B %CC %CS %tsc %ac/%fc/ %bc/%sc/%rc %sq/%bq %hr %hs %{+Q}r %[unique-id]"
 
    filter spoe engine modsecurity config /etc/haproxy/spoe-modsecurity.conf
    http-request deny if { var(txn.modsec.code) -m int gt 0 }
 
 
backend spoe-modsecurity
    mode tcp
    balance roundrobin
    timeout connect 5s
    timeout server 3m
    server modsec1 127.0.0.1 :12345
```
- Validate file HAProxy config
```
$ haproxy -c -f haproxy.cfg
Configuration file is valid
```
- Đăng ký modsec và haproxy với systemd và chạy
```
$ sudo systemctl enable --now modsecurity
```
## Check WAF Operation
- Log ModSec sẽ được lưu trong file audit log. Ta có thể đọc file audit log để kiểm tra xem CRS đã hoạt động chính xác chưa. Đường dẫn audit log có thể được thay đổi trong file modsecurity.conf
- Sau khi chỉnh sửa file configuration, luôn phải restart ModSec service
```
$ sudo vim /etc/modsecurity/modsecurity.conf
SecAuditLogType Serial
SecAuditLog /var/log/modsecurity/modsec_audit.log
SecAuditLogFormat JSON
$ sudo mkdir -p /var/log/modsecurity/
$ sudo systemctl restart modsecurity
```
- Thử gọi một bad request và check audit log
```
$ curl -I https://example.com/?../etc/passwd
$ cat /var/log/modsecurity/modsec_audit.log
```
![image-1](https://raw.githubusercontent.com/nghiahl-sysadmin/haproxy-waf/main/images/image-1.png)
- Có thể dùng jq để xem log dưới dạng json
```
$ cat /var/log/modsecurity/modsec_audit.log | jq
```
![image-1](https://raw.githubusercontent.com/nghiahl-sysadmin/haproxy-waf/main/images/image-2.png)
- Ở cấu hình default, ModSec chỉ chạy với mode DetectionOnly, là phát hiện các truy cập bất thường và log lại chứ không xử lý.
- Giờ ta sẽ enable Rule để block mọi truy cập bất thường.
```
$ sudo vim /etc/modsecurity/modsecurity.conf
SecRuleEngine On
```
- Truy cập vào web bằng một URL đáng ngờ và ta sẽ bị chặn lại
![image-3](https://raw.githubusercontent.com/nghiahl-sysadmin/haproxy-waf/main/images/image-3.png)
