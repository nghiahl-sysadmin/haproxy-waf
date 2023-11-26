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
Cài đặt một số dependencies phục vụ cho quá trình build các packages:
```
$ sudo apt-get -y install build-essential doxygen valgrind libyajl-dev libgeoip-dev liblmdb-dev liblua5.3-dev libpcre2-dev libxml2-dev libcurl4-openssl-dev libfuzzy-dev libevent-dev libpcre3-dev libssl-dev libsystemd-dev jq
```
## 3. Cài đặt
### Cài đặt ModSecurity
```plaintext
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
```plaintext
$ git clone https://github.com/FireBurn/spoa-modsecurity.git
$ cp -r /usr/local/modsecurity/include/modsecurity spoa-modsecurity/include
$ cd spoa-modsecurity
$ sed -i 's/ModSecurity-v3.0.5\/INSTALL\/usr\/local\/modsecurity\/include/\/usr\/local\/modsecurity\/include\/modsecurity/g' Makefile
$ sed -i 's/ModSecurity-v3.0.5\/INSTALL\/usr\/local\/modsecurity\/lib/\/usr\/local\/modsecurity\/lib/g' Makefile
$ make
$ sudo make install
```
