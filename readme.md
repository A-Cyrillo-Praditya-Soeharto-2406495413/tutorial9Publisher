
7. 
a.**How much data your publisher program will send to the message broker in one run?**

Publisher mengirimkan 5 pesan dalam satu kali proses. Setiap UserCreatedEventMessage berisi:
user_id: sebuah string (1 karakter untuk ID 1-5)
user_name: sebuah string (contoh: "2406495413-Amir" = 15 karakter)
Dengan serialisasi Borsh, setiap string memiliki awalan panjang (length prefix) sebesar 4-byte. Kira-kira:
user_id: 5 byte
user_name: 19 byte
Per pesan: 24 byte (hanya body)
5 pesan: 120 byte data aplikasi

Jika menyertakan overhead protokol AMQP (header pesan, properti pengiriman, dll.), total transfer jaringan kira-kira bisa mencapai 500-1000 byte per proses.

**b. The URL `amqp://guest:guest@localhost:5672` is the same as in the subscriber program, what does it mean?**

URL AMQP yang sama tersebut berarti baik program publisher maupun subscriber terhubung ke instans RabbitMQ broker yang sama yang berjalan di mesin lokal. Secara spesifik:
amqp:// — Protokol: AMQP (Advanced Message Queuing Protocol)
guest:guest — Kredensial: username dan password untuk autentikasi
localhost:5672 — Lokasi: RabbitMQ broker yang berjalan pada port 5672 di mesin lokal

Kedua program berbagi message broker yang sama, sehingga pesan yang diterbitkan (published) oleh publisher dapat diterima dan diproses (consumed) oleh subscriber pada infrastruktur queue/exchange yang sama.

![RabbitMQ](asset/RabitMQ.png)

![subscriber console](asset/subscriberconsole.png)
subscriber get the message from publisher

![publisher console](asset/publisherconsole.png)
publisher send the message to subscriber

![rabit mq event spike](asset/RabitMQspike.png)
Setiap kali saya menjalankan perintah cargo run di publisher, program akan mengirimkan message ke RabbitMQ. Spike tersebut menandakan bahwa Publisher berhasil melakukan koneksi dan mengirimkan data ke Exchange pada saat tersebut. Semakin sering perintah dijalankan, semakin tinggi juga puncak lonjakan pada grafik.

![publisher several times](asset/publisherseveral.png)
Hal ini terjadi karena Publisher mengirimkan pesan, tetapi tidak ada Subscriber yang mengambil/memproses pesan tersebut, atau Subscriber memprosesnya lebih lambat daripada kecepatan kirim. Angka tersebut merepresentasikan akumulasi pesan yang menumpuk. Angka tersebut merepresentasikan akumulasi pesan yang menumpuk. Saat menjalankan Publisher, Publisher mengirimkan banyak request (dalam mesin saya sekitar 16 request) dalam waktu yang sangat singkat secara bersamaan. Karena Consumer tidak dapat mengimbangi kecepatan pengiriman Producer, sisa pesan yang belum diproses tidak dibuang, melainkan ditampung sementara di dalam queue RabbitMQ.

![subscriber several](asset/subscriberseveral.png)
karena saya menjalankan 3 subscriber secara bersamaan dan sistem ini menerapkan konsep Competing Consumers. Beban kerja memproses antrean didistribusikan ke tiga subscriber tersebut secara paralel. Karena ada lebih banyak subscriber yang "mengambil" pesan dari antrean di waktu yang sama, tumpukan pesan diproses lebih cepat dan antrean lebih cepat kosong dibandingkan jika hanya menggunakan 1 subscriber seperti sebelumnya.

Menambahkan pengaturan prefetch_count = 1 pada subscriber. Dengan prefetch count, RabbitMQ hanya akan mendistribusikan pesan baru ke subscriber yang sedang tidak sibuk (sudah selesai memproses pesan sebelumnya).