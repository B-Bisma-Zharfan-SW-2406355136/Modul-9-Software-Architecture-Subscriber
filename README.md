<ol>
<li>What is amqp?
    <br>
    amqp adalah singkatan dari Advanced Message Queuing Protocol, yaitu sebuah protokol komunikasi yang digunakan untuk mengirim dan menerima pesan antara aplikasi atau sistem yang berbeda. AMQP dirancang untuk menyediakan komunikasi yang andal, aman, dan terukur antara aplikasi yang berjalan di lingkungan yang berbeda, seperti cloud, on-premises, atau hybrid.
    </br>
</li>
<br>
<li>What does it mean? guest:guest@localhost:5672, what is the first guest, and what  is the second guest, and what is localhost:5672 is for?  
    <br>
    'guest:guest@localhost:5672 adalah format URL yang digunakan untuk mengakses broker AMQP. Berikut adalah penjelasan dari setiap bagian:
    <ul>
        <li><strong>guest:guest</strong> - Ini adalah kredensial login yang digunakan untuk mengakses broker AMQP. 'guest' adalah nama pengguna (username) dan 'guest' juga merupakan kata sandi (password). Dalam banyak kasus, ini adalah kredensial default yang digunakan untuk mengakses broker AMQP, tetapi sebaiknya diganti dengan kredensial yang lebih aman dalam lingkungan produksi.</li>
        <li><strong>localhost:5672</strong> - Ini menunjukkan alamat dan port di mana broker AMQP berjalan. 'localhost' berarti broker AMQP berjalan di mesin lokal (komputer yang sama), dan '5672' adalah port default yang digunakan oleh broker AMQP untuk menerima koneksi. Jika broker AMQP berjalan di mesin lain atau menggunakan port yang berbeda, Anda perlu mengganti bagian ini sesuai dengan konfigurasi broker Anda.</li>
    </ul>
</li>
</ol>

![slow](assets/images/subscriber_simulate_slow.png)

Jumlah total pesan yang mengantri adalah sampai 2 karena publisher mengirim pesan lebih cepat daripada subscriber dapat memprosesnya. Subscriber memiliki jeda 1 detik (thread::sleep) untuk setiap pesan, sehingga ketika subscriber sedang memproses satu pesan, publisher sudah mengirimkan banyak pesan baru secara bersamaan, menyebabkan pesan-pesan tersebut menumpuk sementara di queue sebelum akhirnya diproses satu per satu oleh subscriber.

![faster](assets/images/subscriber_simulate_faster.png)

![faster 2](assets/images/subscriber_simulate_faster_2.png)

Ketika menjalankan beberapa subscriber sekaligus, spike pada grafik RabbitMQ turun lebih cepat dibandingkan hanya dengan satu subscriber. Hal ini terjadi karena pesan-pesan di queue diproses secara paralel oleh beberapa subscriber secara bersamaan (load balancing). Misalnya dengan 3 subscriber, jika publisher mengirim 5 pesan, masing-masing subscriber akan mendapat bagian pesan yang berbeda sehingga waktu pemrosesan total menjadi ~3x lebih cepat dibanding hanya 1 subscriber.

Setelah memperhatikan kode pada *publisher* dan *subscriber*, saya menemukan beberapa area yang perlu diperbaiki (*bad practices*):

1. **Penamaan Variabel yang Menyesatkan:** Di dalam *subscriber*, deklarasi `let ten_milis = time::Duration::from_millis(1000);` sangat membingungkan. Angka 1000 milidetik sebenarnya adalah 1 detik, bukan 10 milidetik.
2. **Variabel yang Tidak Terpakai:** Variabel `let now = time::Instant::now();` dideklarasikan tetapi tidak pernah digunakan sama sekali (misalnya untuk menghitung lama waktu eksekusi).
3. **Busy Waiting yang Menguras CPU:** Kode `loop {}` di akhir fungsi `main` pada *subscriber* menciptakan perulangan tak terbatas yang kosong. Ini akan memaksa CPU bekerja 100% tanpa alasan yang jelas. Jauh lebih baik menggunakan `std::thread::park()` atau menambahkan jeda waktu singkat (seperti `thread::sleep`) di dalam *loop* tersebut agar CPU bisa "bernapas".
4. **Penanganan Error yang Diabaikan (Silent Failure):** Penggunaan `_ = listener.listen(...)` dan `_ = p.publish_event(...)` berarti program sengaja mengabaikan potensi *error*. Jika terjadi kegagalan (misalnya koneksi terputus), program akan diam saja tanpa memberikan pesan *error* apa pun di terminal.
5. **Konfigurasi Hardcode (Magic Strings):** Teks seperti URL koneksi AMQP (`"amqp://guest:guest@localhost:5672"`) dan nama *event* (`"user_created"`) ditulis langsung di dalam kode di banyak tempat. Sebaiknya teks-teks ini dipindahkan ke dalam *environment variables* (file `.env`) atau variabel konstanta agar lebih rapi dan menghindari *typo*.