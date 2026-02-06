# Kimia Farma Big Data Analytics : PBI Rakamin
Pada _project_ ini, Kimia Farma memiliki data penjualan dari tahun 2020 hingga 2023 yang dianalisis untuk memperoleh insight bisnis yang relevan. Analisis dilakukan dengan menyusun tabel analisa yang terdiri dari tren penjualan dari waktu ke waktu, jumlah transaksi, profitabilitas, serta performa cabang dan wilayah berdasarkan rating dan volume transaksi. Hasil analisis ini diharapkan dapat memberikan gambaran menyeluruh mengenai performa bisnis dan membantu pengambilan keputusan yang lebih tepat. Untuk itu, pada tugas akhir ini dilakukan pembuatan dashboard menggunakan Google Looker Studio dari pembentukan query melalui BigQuery.
## Dataset
Terdapat 4 dataset utama yang digunakan mendukung evaluasi performa bisnis Kimia Farma secara menyeluruh.
- **kf_final_transaction** berisi data transaksi penjualan yang mencakup informasi terkait penjualan, baik customer yang melakukan transaksi hingga rating transaksi.
- **kf_inventory** memuat data ketersediaan stok dari beberapa produk.
- **kf_kantor_cabang** berisi informasi mengenai cabang Kimia Farma, seperti lokasi hingga rating cabang di branch berbeda.
- **kf_product** menyediakan informasi produk, termasuk detail harga, nama dan jenis produk.
## Tahapan 1: Tabel Analisa dengan BigQuery
Pada tahapan ini dilakukan impor untuk keempat dataset utama. Selanjutnya, dilakukan proses agregasi data guna menyusun tabel analisis yang mampu memberikan gambaran mengenai penamaan variabel, melakukan perhitungan untuk gross laba dalam bentuk persentase, menghitung nett sales dan profit, serta rating konsumen untuk transaksi.
```
CREATE TABLE `kimia_farma.tabel_analisa` AS
SELECT
  transaksi.transaction_id,
  transaksi.`date`,
  transaksi.branch_id,
  cabang.branch_name,
  cabang.kota,
  cabang.provinsi,
  cabang.rating AS rating_cabang,
  transaksi.customer_name,
  produk.product_id,
  produk.product_name,
  produk.price AS actual_price,
  transaksi.discount_percentage,
```
Dibuat tabel dengan memilih beberapa variabel yang dibutuhkan. Berikutnya, dilakukan perhitungan untuk menemukan persentase gross laba. Perhitungan ini menggunakan query CASE apabila dalam perhitungannya memerlukan persyaratan khusus sebagai berikut.
```
Persentase Gross Laba
  CASE
    WHEN produk.price <= 50000 THEN 0.1
    WHEN produk.price > 50000 AND produk.price <= 100000 THEN 0.15
    WHEN produk.price > 100000 AND produk.price <= 300000 THEN 0.2
    WHEN produk.price > 300000 AND produk.price <= 500000 THEN 0.25
    ELSE 0.3
  END AS persentase_gross_laba,
```
Nett sales adalah total pendapatan dari penjualan setelah dikurangi potongan berupa diskon. Perhitungan untuk menghasilkan nett sales yakni dengan mengalikan harga pada kf_produk yang telah tersimpan sebagai 'produk' dengan mengurangi nilai 1 dan discount_percentage yang tersimpan di kf_transaksi. Karena data yang tersimpan sudah dalam bentuk desimal, maka tidak perlu dilakukan pembagian 100 setelahnya.
```
Hitung Nett Sales
  produk.price * (1 - transaksi.discount_percentage) AS nett_sales,
```
Nett profit adalah jumlah keuntungan akhir yang diperoleh oleh Kimia Farma. Perhitungan ini dilakukan dengan mengalikan perhitungan nett sales dengan persentase gross laba. 
```
Hitung Nett Profit
(produk.price * (1 - transaksi.discount_percentage)) *
  CASE
    WHEN transaksi.price <= 50000 THEN 0.1
    WHEN transaksi.price > 50000 AND transaksi.price <= 100000 THEN 0.15
    WHEN transaksi.price > 100000 AND transaksi.price <= 300000 THEN 0.2
    WHEN transaksi.price > 300000 AND transaksi.price <= 500000 THEN 0.25
    ELSE 0.3
  END AS nett_profit,
```
```
Rating Konsumen Terhadap Transaksi
  transaksi.rating AS rating_transaksi,
```
Sementara itu, digunakan LEFT JOIN untuk memunculkan dataset seperti transaksi, kantor cabang, dan produk.
```
FROM `kimia_farma.Transaksi` AS transaksi
LEFT JOIN `kimia_farma.Cabang` AS cabang
  ON transaksi.branch_id = cabang.branch_id
LEFT JOIN `kimia_farma.Produk` AS produk
  ON transaksi.product_id = produk.product_id;
```
### Hasil Tabel Analisa

### Dashboard Data Penjualan Kimia Farma
https://lookerstudio.google.com/s/gTDR0g7HlpA
