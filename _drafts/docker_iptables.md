# Docker iptables 策略解析



docker 在iptables建立四个过滤链，分别为

1. DOCKER
2. DOCKER-USER
3. DOCKER-ISOLATION-STAGE-1
4. DOCKER-ISOLATION-STAGE-2

DOCKER-ISOLATION-STAGE-1负责处理来自宿主机网卡的网络请求

DOCKER-ISOLATION-STAGE-2负责对宿主机多个网卡之间的互相访问进行隔离

DOCKER负责处理，直接访问DOCKER容器IP端口的通过策略

