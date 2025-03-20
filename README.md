## Modul 6 

1. server single threaded

dalam membuat contoh server singgle threaded disini main.rs memiliki 2 fungsi. fungsi utama dan fungsi untuk mengatasi koneksi. selain itu ada std yang berfungsi untuk impor modul-modul yang diperlukan. dalam fungsi main terdapat listener untuk menangkap koneksi. listener terdiri dari TcpListener::bind("127.0.0.1:7878") yang mendengarkan koneksi dari alamat IP 127.0.0.1 port 7878 dan .unwrap() untuk membongkar apa yang dikembalikan bind. 

kunci utama dari single threadednya terletak pada for stream in listener.incoming() di fungsi main. pada loop ini koneksi yang masuk di urus satu per satu secara berurutan.

pada handle_connection terdapat buf_reader untuk referensi ke stream yang kemudian nantinya akan dibaca baris per baris (.lines()), diubah ke string (.map(|result| result.unwrap())), diambil baris sampai baris kosong(.take_while(|line| !line.is_empty())), kemudian di kumpulkan (.collect()).

pada commit ini belum ada response yang menangani request. 


2. mengembalikan html

![html](/img/2_html.png)
masih menggunakan single-threaded hanya saja dia mengembalikan html sebagai response dari requestnya. 
jika kita terjemahkan kodenya, status_line berfungsi untuk menentukan status yang diberikan response. contents membaca isi file "hello.html" menjadi string. length menghitung panjang isi file. response berisi respon httpnya, diformat sedemikian.
terakhir, stream akan mengembalikan response yang sudah dibuat ke klien.
tidak lupa terdapat file baru "hello.html" yang berisi html page responnya.  


3. validasi request dan respon selektif

![error](/img/3_error.png)
pada commit ini, tidak hanya mengembalikan response dan halaman html tetapi juga memberikan error untuk request url yang belum ditangani.
dengan conditional if untuk menangani request url biasa (http://127.0.0.1:7878/) dan memberikan response sukses (200 OK) dengan isi konten file hello.html. 
kemudian conditional else untuk menangani request lainnya dengan interpretasi url tidak ditemukan. response yang dikembalikan adalah error 404 NOT FOUND. selebihnya seperti biasa pada tiap conditional ada stream untuk mengembalikan response ke klien melalui koneksi TCP.


4. simulasi respon lambat

loading 10s sleep
![loading](/img/4_loading_sleep.png)

setelah loading
![after loading](/img/4_sleeppage.png)
pada commit ini, tujuannya adalah membuat delay sebelum penanganan aktual. 
simulasi dibuat pada url /sleep. pada responsenya diberikan thread::sleep(Duration::from_secs(10)); yang menjeda eksekusi thread selama 10 detik. 
sisanya seperti commit sebelumnya pada url biasa / responsenya 200 OK. selain kedua url responsenya 404 NOT FOUND. 
pada commit ini juga masih menggunakan single threaded saja. 

5. server multithreaded 

![multithreaded](/img/5_multithreaded.png)
pada commit ini server multithreaded dibuat dengan menggunakan ThreadPool. awalnya dibuat 4 ThreadPool. pada setiap stream yang ditangkap, pool.execute akan distribusikan tugas ke thread-thread yang ada. 
secara garis lib.rs menangani pembagian pekerjaan. terdapat implementasi ThreadPool dan Worker. ThreadPool menerima tugas dan membagikannya ke pelayan. Tiap pelayan yang ada menjalankan tugasnya masing-masing. 
saluran mpsc berperan sebagai antrean dengan konsep FIFO (First In, First Out).

#### Bonus

function improvement dengan memanfaatkan while let daripada loop + unwrap di implementasi worker. 
pada kode ini, lock dipastikan dipegang selama diperlukan dan dilepaskan saat selesai. kemudian jika channel ditutup kode optimisasi tidak akan panik. channel dapat ditutup dengan baik. 