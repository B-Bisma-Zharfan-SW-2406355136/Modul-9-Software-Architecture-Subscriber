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
