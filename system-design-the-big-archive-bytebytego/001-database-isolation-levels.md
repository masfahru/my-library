# Database Isolation Levels

Pada pembuatan aplikasi yang memiliki fitur transaksi (seperti marketplace, perbankan dan sejenisnya), perlu menghindari terjadinya race-condition. Untuk itu, kita perlu memahami database isolation levels.

Race-condition pada database adalah keadaan beberapa transaksi mengakses database yang sama secara asinkron ataupun paralel sehingga membuat anomali transaksi.

Beberapa anomali transaksi database seperti yang dijelaskan oleh [Fauna](https://fauna.com/blog/introduction-to-transaction-isolation-levels) :

1. Lost-update anomaly
2. Dirty-write anomaly
3. Dirty-read anomaly
4. Non-repeatable read anomaly
5. Phantom read anomaly
6. Serialization anomaly

## Anomaly

Untuk mempelajari terjadinya anomaly, kita gunakan permisalan kasus sederhana. Andaikan terdapat 2 buah table, yakni inventory dan orders, sebagai berikut:

_Tabel inventory_:
| id | name | stock |
| - | - | - |
| 1 | Sabun | 40 |

_Tabel orders_:
| id | name | qty |
| - | - | - |
|  |  |  |

Dengan langkah transaksi sebagai berikut:

1. Baca nilai stock lama
2. Update stock dengan nilai baru yang dikurangi 1
3. Tambahkan order baru ke tabel orders

### Lost-update anomaly

Andaikan terdapat 2 transaksi (A dan B) yang terjadi secara bersamaan, sehingga dari langkah (1) kedua transaksi membaca stock yang sama: 40, langkah (2) kedua transaksi mengupdate stock dengan nilai baru yang dikurangi 1: 39, dan langkah (3) kedua transaksi menambahkan order baru ke tabel orders, sehingga databasenya menjadi seperti ini:

_Tabel inventory_:
| id | name | stock |
| - | - | - |
| 1 | Sabun | 39 |

_Tabel orders_:
| id | name | qty |
| - | - | - |
| 1 | Sabun | 1 |
| 2 | Sabun | 1 |

Anomali di atas disebut dengan lost-update anomaly.

### Dirty-write anomaly

Andaikan terdapat 2 transaksi (A dan B) yang hampir terjadi secara bersamaan, namun terjadi pembatalan pada transaksi A. Kurang lebih kejadiannya seperti ini:

1. Transaksi (A) membaca stock: 40
2. Transaksi (A) mengupdate stock: 39, secara bersamaan terjadi transaksi (B) yang membaca stock: 39
3. Transaksi (B) mengupdate stock: 38, ternyata transaksi (A) dibatalkan karena tidak bisa membuat order (mungkin saldo kurang), terjadilah REVERT pada transaction A sehingga stock kembali menjadi 40
4. Transaksi (B) melanjutkan menulis order baru ke tabel orders.

sehingga databasenya menjadi seperti ini:

_Tabel inventory_:
| id | name | stock |
| - | - | - |
| 1 | Sabun | 40 |

_Tabel orders_:
| id | name | qty |
| - | - | - |
| 1 | Sabun | 1 |

Anomali di atas disebut dengan dirty-write anomaly.

### Dirty-read anomaly

Dirty-read anomaly adalah kejadian di mana transaksi membaca temporary state dari database. Kejadian Transaksi (B) membaca stok padahal transaksi (A) belum selesai pada contoh di atas merupakan contoh dari dirty-read anomaly. 

### Non-repeatable read anomaly

Non-repeatable read anomaly terjadi karena pembacaan database saat update data ketika transaksi berlangsung, hal ini membuat hasil pembacaan tidak konsisten.

Semisal kita melakukan read data stock ketika berlangsung transaksi (A) langkah (2)mengupdate stock: 39, maka hasil read data stocknya adalah 39. Namun transaksi (A) dibatalkan sehingga stock jika dibaca kembali hasilnya 40.

### Phantom read anomaly

Misalkan terdapat transaksi yang memiliki langkah :

1. Cari maximum order quantity.
2. Cari rerata order quantity 

Ketika langkah 1 selesai, secara bersamaan terjadi transaksi lain yang membuat order dengan quantity besar sehingga nilai rerata order dari langkah 2 menjadi tidak logis karena adanya penambahan data order tersebut. Kejadian ini disebut dengan Phantom read anomaly.

### Serialization anomaly

Misalkan terdapat perusahaan dengan jumlah pegawai sebanyak 9 orang yang digaji perbulan U$D 1000 dan anggaran kantor sebesar U$D 10000. 

Boss ingin memberi bonus 10% pada bulan ini dengan syarat total gaji yang dibayarkan tidak melebihi anggaran, Ini mungkin terjadi karena setelah menambah bonus 10%, total dana untuk semua pegawai adalah 9\*1000\*1.1 = 9900.

Di sisi lain, HRD ingin menambah pegawai dengan syarat penambahan karyawan baru masih dalam cakupan anggaran bulanan. Ini mungkin terjadi karena setelah menambah karyawan, total dana untuk semua pegawai adalah 10\*1000 = 10000.

Nah apa yang terjadi jika keputusan Boss dan HRD dilakukan secara bersamaan, maka Boss dan HRD akan mendapati informasi anggaran dan jumlah karyawan yang sama, dan keputusan keduanya adalah valid. Namun pada kenyataannya, terjadi penambahan karyawan dan juga penambahan bonus yang akan membuat total gaji yang dibayarkan melebihi anggaran, yakni sebesar 10\*1000\*1.1 = 11000.

## Database Isolation

Beberapa database memiliki fitur untuk mengatasi database anomaly, fitur tersebut disebut dengan database isolation. Database Isolation adalah kemampuan database untuk menjalankan transaksi seolah-olah tidak terjadi transaksi yang lain.

Pada situs PostgreSQL (Transaction Isolation)[https://www.postgresql.org/docs/current/transaction-iso.html] dijelaskan beberapa tipe database isolation yang mampu mengatasi beberapa kondisi anomali, seperti pada tabel berikut ini:

| Isolation Level | Dirty Read | Nonrepeatable Read | Phantom Read | Serialization Anomaly
| - | - | - | - | - |
| Read uncommitted | Possible | Possible | Possible | Possible |
| Read committed | Not Possible | Possible | Possible | Possible |
| Repeatable read | Not Possible | Not possible | Possible | Possible |
| Serializable | Not Possible | Not possible | Not Possible | Not Possible |

Namun pada PostgreSQL, hanya terdapat 3 level isolation, yakni **Read committed**, **Repeatable read**, dan **Serializable**.

Cara set isolation pada transaksi PostgreSQL bisa cek tautan berikut ini: [PostgreSQL SET TRANSACTION](https://www.postgresql.org/docs/current/sql-set-transaction.html)
