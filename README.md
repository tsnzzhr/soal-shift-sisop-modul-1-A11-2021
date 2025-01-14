# Penjelasan dan Penyelesaian Soal Shift Modul 1
## Kelompok A11
NRP              | Nama
-----------------|-----------
05111940000032   | Tsania Az Zahra
05111940000153   | Erza Janitradevi Nadine
05111940000220   | Marsa Aushaf Rafi



## No. 1

Mengoutput status info/error beserta pesan dan usernamenya, Menghitung banyak error setiap pesannya, Membuat 2 CSV file yaitu `error_message.csv` dan `user_statistic.csv` dengan input beberapa data spesifik dalam ``syslog.log`` yang didapat dari no *1a,1b, dan 1c*

* Import bash
`#!/bin/bash`


**GREP** : `grep` atau global-regular-expression (regex) yang mana command ini berfungsi untuk mencocokkan data berupa karakter maupun angka yang terdapat di dalam data. grep sangat tepat digunakan untuk mencari pola dari suatu data.

**PIPE** : `pipe` atau dinotasikan dengan `|` adalah command untuk menyatakan redirection atau statement dimana sebelum `|` akan menjadi input untuk statement setelah `|`.

### Narasi Soal :
Ryujin baru saja diterima sebagai IT support di perusahaan Bukapedia. Dia diberikan tugas untuk membuat laporan harian untuk aplikasi internal perusahaan, ticky. Terdapat 2 laporan yang harus dia buat, yaitu laporan daftar peringkat pesan error terbanyak yang dibuat oleh ticky dan laporan penggunaan user pada aplikasi ticky. Untuk membuat laporan tersebut, Ryujin harus melakukan beberapa hal berikut:

## 1a
### Soal :
(a) Mengumpulkan informasi dari log aplikasi yang terdapat pada file syslog.log. Informasi yang diperlukan antara lain: jenis log (ERROR/INFO), pesan log, dan username pada setiap baris lognya. Karena Ryujin merasa kesulitan jika harus memeriksa satu per satu baris secara manual, dia menggunakan regex untuk mempermudah pekerjaannya. Bantulah Ryujin
membuat regex tersebut.
### Jawab :
guna mempermudah penggunaan dan fleksibilitas nama file syslog.log, maka digunakan sebuah variabel file untuk menampung nama file syslog.log
```
file='syslog.log'

error=$(cat $file | grep 'ERROR')
info=$(cat $file | grep 'INFO')

echo "$error" | while read row;
do
status=$(echo $row | grep -oP 'INFO|ERROR')
message=$(echo $row | grep -oP '(?<=[ERROR|INFO]\s)(.*)(?<=[a-z] )')
username=$(echo $row | grep -oP '(?<=\()(.*)(?=\))')
ec="$status,$message,$username"
echo $ec
done

echo "$info" | while read row;
do
status=$(echo $row | grep -oP 'INFO|ERROR')
message=$(echo $row | grep -oP '(?<=[ERROR|INFO]\s)(.*)(?<=[a-z] )')
username=$(echo $row | grep -oP '(?<=\()(.*)(?=\))')
ec="$status,$message,$username"
echo $ec
done
```
pada variabel error dan info digunakan pipe dan regular-expression dimana nilai dari variabel error dan info didapat dari isi file syslog.log yang kemudian dijadikan input untuk regex/grep setelahnya. dimana variabel ini pada akhirnya akan mengembalikan nilai dari seluruh baris yang mengandung kata ERROR/INFO. 

kemudian untuk mencari jumlah error/info digunakan lagi regex/grep dengan memisahkan output berdasarkan variabel info/error yang sudah didapat sebelumnya. dengan menggunakan pipe dan looping yang membaca setiap baris dari variabel info/error kemudian didapat status error/info, pesan error/info, dan usernamenya. `-oP` adalah gabungan dari command 'only-match' dan 'Perl' sebagai standar grep yang keduanya berfungsi untuk mengambil karakter yang HANYA COCOK dengan karakter regex yang disebutkan. jadi tidak akan diambil semua isi barisnya.


### 1b
### Soal :
(b) Kemudian, Ryujin harus menampilkan semua pesan error yang muncul beserta jumlah kemunculannya.
### Jawab :
```
pesan=$(echo "$error" | grep -Po '(?<=ERROR\s)(.*)(?<=[a-z] )')
hitung_pesan=$(echo "$pesan" | sort | uniq -c | sort -nr)

#echo $pesan
echo $hitung_pesan
```
dari variabel error yang sudah didapat di nomor sebelumnya pada bagian ini variabel error digunakan lagi untuk mencari pesan yang terdapat di setiap barisnya dengan mengcapture setiap baris yang datang setelah kata 'ERROR' diikuti 1 spasi hingga menemui akhir string sebelum spasi yang datang sesudah akhir string pesan.

pada bagian hitung pesan, pesan yang sudah diambil dari variabel pesan akan diurutkan sesuai kesamaan stringnya kemudian dihitung dengan `-c` bersamaan dengan mengoutputkan salah satu perwakilan string pesan yang dihitung. sort `-nr` [number reversed] berfungsi sebagai yang mengurutkan berdasarkan angka secara ascending tetapi mengalami reverse sehingga mengakibatkan terurut secara descending.

### 1c
### Soal :
(c) Ryujin juga harus dapat menampilkan jumlah kemunculan log ERROR dan INFO untuk setiap user-nya.
### Jawab :
```
user=$(cat $file | grep -Po '(?<=\()(.*)(?=\V)' | sort | uniq)

for line in $user;
do
err_count=$(cat $file | grep 'ERROR' | grep -wc $line )
info_count=$(cat $file | grep 'INFO' | grep -wc $line )
echo "$line,$info_count,$err_count"
done
```
variabel user berfungsi untuk mencari seluruh nama user dengan menghilangkan duplikasi sehingga akan dimunculkan setiap user hanya 1 nama dan sudah disortir/diurutkan. kemudian pada bagian looping dicek untuk setiap usernya untuk dihitung tiap error dan info yang tertulis dalam file syslog-nya. masih menggunakan regex dan pipe kemudian ditambahkan command `-wc` yang berarti word dan count dimana variabel line akan dihitung sebagai 1 string sehingga tidak ada backtracking/look forward yang mengakibatkan hitungan mencakup nama user lain serta count seperti penjelasan sebelumnya berfungsi sebagai pencacah.

### 1d
### Soal :
(d) Semua informasi yang didapatkan pada poin b dituliskan ke dalam file error_message.csv dengan header Error,Count yang kemudian diikuti oleh daftar pesan error dan jumlah kemunculannya diurutkan berdasarkan jumlah kemunculan pesan error dari yang terbanyak.
Contoh:
```
Error,Count
Permission denied,5
File not found,3
Failed to connect to DB,2
```
### Jawab :
```
echo 'Error,Count' > error_message.csv
echo "$hitung_pesan" | while read row;
do
error=$(echo $row | cut -d ' ' -f 2-)
count=$(echo $row | cut -d ' ' -f 1)
ec="$error,$count"
echo $ec >> error_message.csv
done
```
`echo` yang pertama berfungsi untuk menambahkan header pada .csv file.
setelahnya terjadi looping dari variabel hitung_pesan dimana setiap pembacaan baris akan diambil variabel error yang berisi pesan error kemudian diberikan command `cut` guna memotong bagian-bagian stringnya dengan `-d` berfungsi sebagai delimiter yang menghentikan pengambilan argumen. pada kasus ini digunakan empty_string karena baris hitung_pesan terdiri dari empty_space dari argumen 1 ke argumen yang lain. kemudian `-f` berarti field yang berguna untuk mengambil nilai argumen. berlaku juga `cut`, delimiter dan field pada variabel count.

variabel ec adalah variabel final yang akan dioutputkan untuk kemudian ditambahkan pada tail file error_message.csv

### 1e
### Soal :
(e) Semua informasi yang didapatkan pada poin c dituliskan ke dalam file user_statistic.csv dengan header Username,INFO,ERROR diurutkan berdasarkan username secara ascending.
```
Username,INFO,ERROR
kaori02,6,0
kousei01,2,2
ryujin.1203,1,3
```
### JAWAB :
```
echo 'Username,INFO,ERROR' > user_statistic.csv
for row in $user;
do
err_count=$(cat $file | grep 'ERROR' | grep -wc $row )
info_count=$(cat $file | grep 'INFO' | grep -wc $row )
echo "$row,$info_count,$err_count" >> user_statistic.csv
done
```
pada bagian ini setiap variabel berguna untuk menghitung banyak error dan info dari setiap user sebagaimana pada poin 1c kemudian dioutputkan secara langsung dan ditambahkan pada tail file user_statistic.csv

### Kendala yang dialami selama mengerjakan nomor 1
![Capture](https://user-images.githubusercontent.com/69724694/113371083-9e472080-938f-11eb-802e-5a5f947cecd1.PNG)
1. terjadi ketidaksamaan output pada nomor 1c dimana hasil perhitungan error dan info tiap user berbeda dari apa yang diharapkan. setelah diperbaiki diketahui bahwa command yang digunakan salah yaitu seharusnya menggunakan `-wc` daripada menggunakan `-oPc` karena jika menggunakan `-oPc` akan menghitung semua yang mengandung semua kata yang mengandung string tiap user. misalnya ac dan jackowen akan terhitung menjadi satu ketika terjadi iterasi pada ac. jadi ketika menggunakan `-wc` string user dimutlakkan menjadi 1 word tanpa karakter backtracking atau look-forward.


## No. 2
Mencari beberapa kesimpulan dari data penjualan ``Laporan-TokoShiSop.tsv`` yang akan dijelaskan pada no *2a, 2b, 2c, dan 2d*. Soal *2a, 2b, 2c, dan 2d* dikerjakan pada script ``soal2_generate_laporan_ihi_shisop.sh``. Hasil pengerjaan soal tersebut ditampilkan pada ``hasil.txt``.
* Import awk
``` #!/bin/bash
    awk '
```
 Untuk mengerjakan soal ini kami menggunakan ``awk`` sehingga pada awal ``shell script`` harus mengimport ``awk`` terlebih dahulu.
 * Field Separator
 ```
    BEGIN	{
    FS="\t"
    }
 ```
 Karena pada file ``Laporan-TokoShiSop.tsv`` tiap kolom dipisahkan dengan *tab* maka menggunakan *FS="\t"*

### Blok BEGIN
### 2a
**Soal**: Steven ingin mengapresiasi kinerja karyawannya selama ini dengan mengetahui **Row ID** dan **Profit Percentage** terbesar (jika hasil profit percentage terbesar lebih dari 1, maka ambil Row ID yang paling besar). Karena kamu bingung, Clemong memberikan definisi dari profit percentage, yaitu:
``Profit Percentage = (Profit ÷ Cost Price) × 100``
Cost Price didapatkan dari pengurangan Sales dengan Profit. (Quantity diabaikan).
* Mencari nilai ``Profit Percentage``
  ```soal2_generate_laporan_ihir_shisop.sh
     pp=($21/($18-$21))*100
  ```
  Berdasarkan rumus untuk mencari **Profit Percentage** yang telah diberikan, value dari kolom **Profit** didapatkan dengan menggunakan **$21**. Sementara itu, untuk mendapatkan **Cost Price** yaitu dengan mengurangi *Sales* yang didapatkan dengan **$18** dan *Profit* yang didapatkan dengan **$21**. Kemudian hasil pembagian dari *Profit* dan *Cost Price* dikalikan dengan 100.
* Mendapatkan nilai ``Profit Percentage`` terbesar
  ```soal2_generate_laporan_ihir_shisop.sh
     if(pp>=max){
        max=pp
        id=$1
     }
  ```
    Membandingkan **Profit Percentage** yang dituliskan dengan variabel *pp* dengan suatu variabel *max* untuk menyimpan nilai terbesar. Apabila nilai *pp* lebih besar dibandingkan dengan nilai *max* maka *Profit Percentage* terbesar akan disimpan di variabel *max*. Kemudian, ``id=$1`` digunakan untuk menyimpan *Row ID* paling besar.
### 2b
**Soal**: Clemong memiliki rencana promosi di Albuquerque menggunakan metode MLM. Oleh karena itu, Clemong membutuhkan daftar *nama customer* pada transaksi tahun 2017 di Albuquerque.
* Mendapatkan ``nama customer`` 
  ```soal2_generate_laporan_ihir_shisop.sh
      if($2~"2017" && $10=="Albuquerque"){
        nama[$7]++
     }
  ```
  Menggunakan *if condition* untuk mengecek apakah customer tersebut melakukan transaksi tahun 2017 di Albuquerque. Untuk mendapatkan angka berupa string 2017 dari kolom ``Order ID`` menggunakan **$2~"2017"**. Kemudian mengecek apakah customer tersebut melakukan transaksi di Albuquerque dari kolom ``City``dengan **$10=="Albuquerque"**. Setelah itu, nama customer akan disimpan di sebuah array ``nama[$7]++`` dengan **$7** adalah kolom ``Customer Name``. Maka nama customer yang memenuhi *if condition* akan tersimpan di array ``nama[Albuquerque]``.
### 2c
**Soal**: TokoShiSop berfokus tiga segment customer, antara lain: **Home Office**, **Consumer**, dan **Corporate**. Clemong ingin meningkatkan penjualan pada segmen customer yang paling sedikit. Oleh karena itu, Clemong membutuhkan segment customer dan jumlah transaksinya yang paling sedikit.
* Menghitung jumlah transaksi masing-masing segment customer
  ```soal2_generate_laporan_ihir_shisop.sh
      if ($8=="Home Office") {
         sc1+=1
      }
      else if ($8=="Consumer") {
         sc2+=1
      }
      else if ($8=="Corporate") {
         sc3+=1
      }
  ```
Menggunakan *if-else condition* untuk menghitung jumlah transaksi masing-masing segment. Segment customer berada pada kolom ``Segment`` yang bisa dituliskan dengan **$8**. Untuk menghitung menggunakan counter, yaitu *sc1* untuk **Home Office**, *sc2* untuk **Consumer**, dan *sc3* untuk **Corporate**. Counter akan bertambah apabila kondisi yang sedang dicek memenuhi salah satu syarat tersebut. Kemudian, untuk mencari nilai terkecilnya akan dijelaskan pada bagian ``Blok END`` **2c**
### 2d
**Soal**: TokoShiSop membagi wilayah bagian (region) penjualan menjadi empat bagian, antara lain: **Central, East, South, dan West**. Manis ingin mencari wilayah bagian
(region) yang memiliki **total keuntungan (profit) paling sedikit** dan **total keuntungan wilayah** tersebut.
* Menghitung total keuntungan masing-masing wilayah bagian (region)
  ```soal2_generate_laporan_ihir_shisop.sh
     if ($13=="Central" || $13=="East" || $13=="South" || $13=="West"){
        #jumlah semua profit
        reg[$13]+=$21
    }
  ```
  Menggunakan *if condition* untuk mengecek string **Central, East, South, dan West** pada kolom ``Region`` yang dituliskan dengan **$13**. Total keuntungan masing-masing region disimpan pada array *reg*. ``reg[$13]+=$21`` artinya *Profit* yang dituliskan dengan **$21** akan dijumlahkan untuk salah satu region. Kemudian, untuk mencari nilai terkecilnya akan dijelaskan pada bagian ``Blok END`` **2d**
 
### Blok END
Pada *awk*, semua fungsi *printf* dimasukkan pada blok ``END``.
* Menampilkan ``Row ID`` dan ``Profit Percentage`` dari soal **2a**
  ```soal2_generate_laporan_ihir_shisop.sh
     printf("Transaksi terakhir dengan profit percentage terbesar yaitu %d dengan persentase %.2f%%",id,max)
  ```
  Menampilkan **Row Id** dan **Profit Percentage** yang telah didapatkan dengan variabel *id* dan *max*. 
* Menampilkan ``nama customer``  dari soal **2b**
  ```soal2_generate_laporan_ihir_shisop.sh
      printf("Daftar nama customer di Albuquerque pada tahun 2017 antara lain :\n")
      for(i in nama){
        printf("%s\n", i)
     }
  ```
    Iterasi semua nilai pada array ``nama`` untuk menampilkan semua nama pada array tersebut. 
* Menampilkan ``Tipe Segment`` yang memiliki jumlah transaksi terkecil dan ``Jumlah Transaksi Segment`` dari soal **2c**
  ```soal2_generate_laporan_ihir_shisop.sh
     if(sc1<sc2 && sc1<sc3){
        printf("Tipe segmen customer yang penjualannya paling sedikit adalah Home Office dengan %d transaksi\n", sc1)
      }
     else if(sc2<sc1 && sc2<sc3){
        printf("Tipe segmen customer yang penjualannya paling sedikit adalah Consumer dengan %d transaksi\n", sc2)
      }
     else if(sc3<sc1 && sc3<sc2){
        printf("Tipe segmen customer yang penjualannya paling sedikit adalah Corporate dengan %d transaksi\n", sc3)
     }
  ```
    Menggunakan *if-else condition* untuk mencari jumlah transaksi terkecil. Terdapat 3 kondisi, yaitu apabila *sc1* adalah yang terkecil, atau *sc2* maupun *sc3*.
* Menampilkan ``Nama Region`` yang memiliki total keuntungan terkecil dan ``Total Keuntungan Region``  dari soal **2d**
  ```soal2_generate_laporan_ihir_shisop.sh
     profit=99999999
    for(i in reg){
    if(profit>reg[i]){
        profit=reg[i]
        nama_reg=i
    }
    }
    printf("Wilayah bagian (region) yang memiliki total keuntungan (profit) yang paling sedikit adalah %s dengan total keuntungan %.2f\n", nama_reg,profit)
  ```
    Iterasi total keuntungan masing-masing region pada array ``reg``, kemudian dibandingkan dengan sebuah variabel *profit* untuk mencari nilai total terkecil. Nama region atau *i* disimpan pada variabel *nama_reg*.

### 2e
* Output soal 2a, 2b, 2c, dan 2d ditampilkan pada file **hasil.txt**
* ```/home/erzajanitra/Downloads/Laporan-TokoShiSop.tsv > hasil.txt```
### hasil.txt
![image](https://user-images.githubusercontent.com/75319371/113247910-1ce78380-92e6-11eb-937c-ad3334282920.png)

### Kendala yang dialami selama mengerjakan nomor 2
1. Pada saat mengerjakan no 2a, variabel yang digunakan untuk menunjukkan hasil *profit percentage* terbesar salah sehingga output menjadi salah. Seharusnya kami menggunakan    variabel *max*, tetapi kami menggunakan variabel *pp*. Selain itu, awalnya kami menggunakan *NR* untuk menampilkan *Row ID* terbesar. Seharusnya kami langsung menyimpan      *Row ID* dimana nilai *profit percentage* terbesar itu ditemukan.
2. Kendala ketika mengerjakan no 2d, yaitu terjadi eror ketika menggunakan *for loop* untuk mencari *profit* terkecil. Karena total profit memiliki jumlah yang sangat besar,    variabel *profit* sebagai pembanding dengan *reg[i]* harus dibuat menjadi 8 digit angka agar tidak mengalami eror.

## No. 3
### 3a
* Mengunduh gambar dari "https://loremflickr.com/320/240/kitten" 
```soal2_generate_laporan_ihir_shisop.sh
    for i in {1..23}
    do 
         wget -a Foto.log "https://loremflickr.com/320/240/kitten" -O "Koleksi_$i.jpg"
    done
```
```wget -a``` digunakan untuk membuat logfile dari mendownload foto dan disimpan pada *Foto.log*. Sedangkan, ``` -O ``` digunakan untuk memberikan nama file untuk masing-masing gambar yang di download.
* Menghapus gambar yang sama tanpa mengunduh gambar lagi
```declare -A koleksi_foto``` untuk membuat array bernama *koleksi_foto*.```shopt -s globstar``` digunakan untuk mencari file gambar yang sama. ```cksm``` untuk mengecek jumlah gambar yang sama. Apabila ada gambar yang sama akan di remove. 
* Rename nama file 
``` j=1
    for i in {1..23} 
        do
        if [ -f Koleksi_$i.jpg ] 
        then
            if [ $i -lt 10 ]
            then
            mv Koleksi_$i.jpg Koleksi_0$j.jpg
            j=$[$j+1]
            else
            mv Koleksi_$i.jpg Koleksi_$j.jpg
            j=$[$j+1]
            fi

        fi

    done
```
Gambar disimpan dengan format nama *Koleksi_XX*, maka mengecek gambar yang telah melewati pengecekan untuk gambar yang sama. Untuk file urutan 1-9, nama file akan diganti menjadi *Koleksi_0$i* kemudian menggunakan variabel *j* untuk menandakan bahwa file tersebut sudah direname. Sedangkan, untuk file urutan 10-23 nama file akan diganti menjadi *Koleksi_$j*.
#### Hasil download gambar kucing
![image](https://user-images.githubusercontent.com/75319371/113500553-16a51180-9549-11eb-8cd0-0c2446cf3f8d.png)
#### Foto.log (log download gambar kucing)
![image](https://user-images.githubusercontent.com/75319371/113498999-56b1c780-953c-11eb-991d-2d9aca6f0258.png)


### 3b
#### Crontab
* Menjalankan script **sehari sekali pada jam 8 malam dari tanggal 1 tujuh hari sekali dan dari tanggal 2 empat hari sekali**
```0 20 1-31/7,2-31/4 * * /bin/bash /home/erzajanitra/shift1soal3/soal3b.sh```

#### soal3b.sh
* Memindahkan gambar ke folder dengan nama tanggal download
```
    bash /home/erzajanitra/shift1soal3/soal3a.sh
    mv *.jpg $fotoKucing
    mv Foto.log $fotoKucing
```
Gambar yang telah didownload dari script **soal3a.sh** dipindahkan ke folder *shift1soal3* dengan nama directory tanggal download  *DD-MM-YYYY*
#### Hasil setelah memindahkan gambar ke folder dengan nama directory tanggal download
![image](https://user-images.githubusercontent.com/75319371/113499024-88c32980-953c-11eb-978b-bf310b0d737f.png)

### 3c
Mengunduh gambar kelinci dari "https://loremflickr.com/320/240/bunny", kemudian gambar tersebut diunduh bergantian per hari dengan gambar kucing dari "https://loremflickr.com/320/240/kitten". 
* Membuat fungsi ```kelinciF()``` dan ```kucingF()``` untuk mengunduh gambar dan menyimpan      gambar pada directory ```"Kelinci_$tanggal"``` untuk kelinci dan ```"Kucing_$tanggal"```
* Mendownload gambar kucing dan kelinci bergantian
```
   dates=$(date +%e)
    if [ $(($dates%2)) -eq 0 ]
        then kucingF
    else kelinciF
    fi
```
* Mengecek apakah hari ini memiliki tanggal ganjil atau genap untuk mengunduh gambar secara bergantian. Apabila tanggal genap maka akan mengunduh gambar kucing dan akan         mengunduh gambar kelinci ketika tanggal ganjil.

### 3d
Membuat script yang akan memindahkan seluruh folder ke zip yang diberi nama “Koleksi.zip” dan mengunci zip tersebut dengan password berupa tanggal saat ini dengan format "MMDDYYYY".
```password=$(date +"%m%d%Y")
    zip -P $password -mr Koleksi.zip $(ls | grep -E "Kelinci_|Kucing_")
```
* ```-P``` password berupa tanggal ketika di zip
* ```-mr``` nama folder zip yaitu *Koleksi.zip*
* ```$(ls | grep -E "Kelinci_|Kucing_")``` folder yang akan di zip adalah "Kelinci_$tanggal" dan "Kucing_$tanggal"
#### Proses zip folder Kelinci dan Kucing
![image](https://user-images.githubusercontent.com/75319371/113499049-e9526680-953c-11eb-9875-9620d27242f1.png)

#### Folder Koleksi.zip
![image](https://user-images.githubusercontent.com/75319371/113405627-54ccf480-93d4-11eb-9eed-6378a3806eb8.png)


### 3e
Membuat koleksinya menjadi zip pada jam 7-18 dan pada hari Senin-Jumat, selain dari waktu yang disebutkan, ia ingin koleksinya ter-unzip dan tidak ada file zip sama sekali.
* ```0 7 * * 1-5 /bin/bash /home/erzajanitra/shift1soal3/soal3d.sh```
* ```0 18 * * 1-5 unzip -P $(date +"%m%d%Y") "Koleksi.zip" && rm "Koleksi.zip" ```

### Kendala yang dialami selama mengerjakan nomor 3
1. **Revisi no 3a** 
    Gambar yang didownload memiliki nomer yang tidak urut. Solusinya dengan mengecek gambar yang telah didownload, mengecek urutan 1-9 dan mengganti nama file menjadi             *Koleksi_0$i* lalu menggunakan variabel *j* untuk menandai bahwa gambar tersebut sudah di rename. Untuk gambar dengan urutan 10-23, nama filenya menjadi *Koleksi_$j*.
2. **Revisi no 3c**
    Awalnya untuk mengunduh gambar kucing dan kelinci secara bergantian kami membandingkan jumlah file kucing dan kelinci yang sudah didownload. Tetapi, karena pada soal         diminta untuk download bergantian tiap hari, kami mengecek apakah tanggal pada hari ini adalah tanggal ganjil atau genap.
3.  Pada saat mengerjakan no 3e, kami salah meletakkan *1-5* pada bagian day(month) dimana artinya akan menjadi tanggal 1-5 pada sebuah bulan. Sementara perintah pada soal       adalah hari senin-jumat, sehingga *1-5* diletakkan di bagian day(week).
4. Pada saat membuat crontab 3b, awalnya kami membuat dua crontab yaitu ```0 20 1-31/7 * *``` dan ```0 20 2-31/4 * *``` karena kami mengira akan eror apabila digabungkan        menjadi satu seperti ini ```0 20 1-31/7,2-31/4 * *```. Tetapi apabila dibuat menjadi dua crontab, gambar yang akan di download akan double pada tanggal 22.
