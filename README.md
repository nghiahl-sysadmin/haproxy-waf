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
