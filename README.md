<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/3/35/Tux.svg" alt="Linux Logo" width="120">
</p>

## ![Lesson](https://img.shields.io/badge/Lesson-Network__Lab-00758F?style=for-the-badge&logo=linux&logoColor=white&labelColor=111827)![Author](https://img.shields.io/badge/Author-Kamil%20Ibragimov-10B981?style=for-the-badge&logo=github&logoColor=white&labelColor=111827)![Date](https://img.shields.io/badge/Date-18.12.2025-F59E0B?style=for-the-badge&logo=calendar&logoColor=white&labelColor=111827)

### 📌 Задание
1. Развернуть инфраструктуру из 7 ВМ (3 роутера, 3 сервера, 1 интернет-шлюз).
2. Реализовать схему сети: Office1, Office2, Central.
3. Настроить статическую маршрутизацию между офисами.
4. Настроить выход в интернет для всех узлов через NAT (inetRouter).

### ✅ Результат
- [x] Стенд развернут (Vagrant + VirtualBox).
- [x] Настройка сети автоматизирована (Ansible).
- [x] Связность между офисами есть.
- [x] Выход в интернет есть (NAT Masquerade).

**Демонстрация маршрутизации (Traceroute до 8.8.8.8 из Office2):**
Видна цепочка: `Office2Router` -> `CentralRouter` -> `InetRouter` -> `NAT`.
```bash
vagrant@office2Server:~$ traceroute 8.8.8.8
traceroute to 8.8.8.8 (8.8.8.8), 30 hops max, 60 byte packets
 1  _gateway (192.168.1.1)  0.258 ms  0.237 ms  0.227 ms
 2  192.168.255.5 (192.168.255.5)  0.352 ms  0.344 ms  0.615 ms
 3  192.168.255.1 (192.168.255.1)  0.661 ms  0.654 ms  0.647 ms
 4  _gateway (10.0.2.2)  0.647 ms  0.619 ms  0.588 ms
 5  * * *
 6  * * *
 7  * * *
 8  10.254.253.26 (10.254.253.26)  3.217 ms  3.207 ms  3.193 ms
 9  192.178.70.14 (192.178.70.14)  2.647 ms  2.567 ms  2.568 ms
10  * * *
11  72.14.233.96 (72.14.233.96)  5.662 ms 72.14.235.226 (72.14.235.226)  2.424 ms 108.170.226.90 (108.170.226.90)  2.393 ms
12  * * 192.178.241.234 (192.178.241.234)  5.549 ms
13  * * *
14  142.251.237.148 (142.251.237.148)  17.755 ms 74.125.253.94 (74.125.253.94)  15.715 ms 72.14.232.86 (72.14.232.86)  20.614 ms
15  142.250.233.27 (142.250.233.27)  18.691 ms 172.253.79.115 (172.253.79.115)  21.012 ms 142.250.56.219 (142.250.56.219)  18.220 ms
16  * * *
17  * * *
18  * * *
19  * * *
20  * * *
21  * * *
22  * * *
23  * * *
24  * * *
25  dns.google (8.8.8.8)  17.215 ms  15.277 ms *
```

### 🧭 Оглавление
- [🧰 Шаг 1 - Инфраструктура](#one)
- [🧰 Шаг 2 - Настройка Ansible](#two)
- [🧰 Шаг 3 - Проверка](#three)

---

<a id="one"></a>
## 🧰 Шаг 1 - Инфраструктура

Развертывание ВМ описано в `Vagrantfile`.
Топология:
* **Office1:** 192.168.2.0/26
* **Office2:** 192.168.1.0/25
* **Central:** 192.168.0.0/28


<a id="two"></a>
## 🧰 Шаг 2 - Настройка Ansible
Конфигурация выполняется через Ansible (`ansible/provision.yml`).
Запуск:
```bash
ansible-playbook ansible/provision.yml
```

Основные задачи плейбука:

**1. Включение IP Forwarding (Routers)**
```yaml
- name: Enable net.ipv4.ip_forward
  sysctl:
    name: net.ipv4.ip_forward
    value: '1'
    state: present
```

**2. Настройка NAT (InetRouter)**
```yaml
- name: Configure NAT on InetRouter
  shell: |
    iptables -t nat -F POSTROUTING
    iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
  when: inventory_hostname == "inetRouter"
```

**3. Статическая маршрутизация (CentralRouter)**
```yaml
- name: Configure routes on CentralRouter
  shell: |
    ip route add 192.168.1.0/24 via 192.168.255.6
    ip route add 192.168.2.0/24 via 192.168.255.10
    ip route add default via 192.168.255.1
  when: inventory_hostname == "centralRouter"
```

<a id="three"></a>
## 🧰 Шаг 3 - Проверка

1. Проверка связности между офисами (с `office2Server` на `office1Server`):
```bash
vagrant ssh office2Server
ping -c 4 192.168.2.130
# 64 bytes from 192.168.2.130: icmp_seq=1 ttl=61 time=0.824 ms (OK)
```

2. Проверка выхода в интернет:
```bash
ping -c 4 8.8.8.8
# 64 bytes from 8.8.8.8: icmp_seq=1 ttl=61 time=17.215 ms (OK)
```
