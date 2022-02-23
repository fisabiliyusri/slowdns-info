# slowdns-info
info slowdns

# Untuk SLOWDNS 

## DNS zone setup

Karena sisi server terowongan bertindak seperti nama otoritatif server, Anda harus memiliki nama domain dan menyiapkan subdomain untuk terowongan. Katakanlah nama domain Anda adalah example.com dan IP server Anda alamatnya adalah 203.0.113.2 dan 2001:db8::2. Pergi ke pencatat nama Anda dan tambahkan tiga catatan baru:
``` 
A 	tns.example.com	points to 203.0.113.2
AAAA	tns.example.com	points to 2001:db8::2 
NS	t.example.com	is managed by tns.example.com
 ```
Label `tns` dan `t` dapat berupa apa saja yang Anda inginkan, tetapi label `tns` tidak boleh menjadi subdomain dari label `t` (ruang itu disediakan untuk isi terowongan), dan label `t` harus pendek (karena ada ruang terbatas yang tersedia dalam pesan DNS, dan nama domain mengambil bagian dari ruang itu).
Sekarang, ketika resolver DNS rekursif menerima kueri untuk nama seperti aaaa.t.example.com, itu akan meneruskan kueri ke server terowongan di 203.0.113.2 atau 2001:db8::2.

#
Pertama, Anda perlu membuat pasangan kunci server yang akan digunakan untuk mengotentikasi server dan mengenkripsi terowongan.
```
./dnstt-server -gen-key -privkey-file server.key -pubkey-file server.pub 
privkey written to server.key 
pubkey written to server.pub
```
Jalankan servernya. Anda perlu memberikan alamat yang akan mendengarkan UDP
Paket DNS (`:5300`), file private key (`server.key`), root dari zona DNS (`t.example.com`), dan alamat TCP yang masuk
contoh:

aliran terowongan akan diteruskan (`127.0.0.1:8000`).
./dnstt-server -udp :5300 -privkey-file server.key t.example.com 127.0.0.1:8000

Server terowongan harus dapat menerima paket di eksternal port 53. Anda dapat mendengarkannya di port 53 secara langsung menggunakan `-udp :53`, tetapi itu membutuhkan program untuk dijalankan sebagai root. Lebih baik menjalankan program sebagai pengguna biasa dan mendengarkannya di port yang tidak memiliki hak istimewa (`:5300` di atas), dan port-forward port 53 ke sana. Di Linux, gunakan ini perintah untuk meneruskan port eksternal 53 ke port localhost 5300:
"

sudo iptables -I INPUT -p udp --dport 5300 -j ACCEPT
sudo iptables -t nat -I PREROUTING -i eth0 -p udp --dport 53 -j REDIRECT --to-ports 5300
sudo ip6tables -I INPUT -p udp --dport 5300 -j ACCEPT
sudo ip6tables -t nat -I PREROUTING -i eth0 -p udp --dport 53 -j REDIRECT --to-ports 5300

"
