
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