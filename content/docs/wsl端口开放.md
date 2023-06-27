### wsl端口开放

### 删除当前所有映射

netsh interface portproxy reset

### 映射windows 192.168.123.95的3000端口到WS2的ip的3000端口：

netsh interface portproxy add v4tov4 listenaddress=192.168.123.95 listenport=9000 connectaddress=172.20.252.166 connectport=9000

### 设置Windows的防火墙，允许监听端口的对内连接

netsh advfirewall firewall add rule name="Open Port 9000 for WSL2" dir=in action=allow protocol=TCP localport=9000

### 显示当前所有映射关系

netsh interface portproxy show all
