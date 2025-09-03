# **Laporan Proyek Machine Learning - Syahel Rusfi Razaba**
*Webtoon Recommender System*

## **Project Overview**
Sistem rekomendasi merupakan komponen penting dalam berbagai platform hiburan digital, termasuk Webtoon, yang menawarkan ribuan judul komik dengan beragam genre. Banyaknya pilihan judul sering kali membuat pengguna kesulitan menemukan komik yang sesuai dengan minat mereka. Oleh karena itu, proyek ini bertujuan membangun sistem rekomendasi berbasis konten (*content-based filtering*) yang dapat membantu pengguna menemukan Webtoon yang relevan berdasarkan kemiripan genre dan ringkasan cerita, sehingga pengalaman pengguna dapat menjadi lebih terarah dan menyenangkan.

## **Business Understanding**
### Problem Statement
- Bagaimana cara membantu pengguna menemukan Webtoon yang sesuai dengan preferensi mereka di antara banyaknya judul yang tersedia?
- Bagaimana cara merekomendasikan Webtoon yang relevan berdasarkan kemiripan konten seperti genre dan ringkasan cerita?

### Goals
- Mengembangkan sistem rekomendasi berbasis konten untuk membantu pengguna menemukan Webtoon yang relevan.
- Memberikan daftar Webtoon yang mirip berdasarkan genre dan ringkasan cerita.

### Solution Approach
- Sistem rekomendasi dibangun menggunakan pendekatan *Content-Based Filtering* yang mengandalkan kemiripan antar konten berdasarkan genre dan ringkasan cerita.

## **Data Understanding**
Dataset yang digunakan adalah Webtoon Dataset yang diperoleh dari [Kaggle](https://www.kaggle.com/datasets/swarnimrai/webtoon-comics-dataset).
### Informasi Dataset
- Jumlah sampel: 569 baris (sebelum cleaning)
- Jumlah fitur: 10 fitur

### Kondisi Dataset
- Tidak terdapat duplikasi data
- Data memiliki 1 *missing value* pada kolom `Writer`
- Data memiliki *outlier* pada fitur numerik (`Likes`, `Rating`, dan `Subscribers`)

### Fitur
| Fitur | Deskripsi |
| ------ | ------ |
| id | ID unik yang mengidentifikasi komik. |
| Name | Nama lengkap komik. |
| Writer | Penulis komik. |
| Likes | Total jumlah Like. |
| Genre | Genre komik. |
| Rating | Rata-rata rating komik dari skala 10. |
| Subscribers | Total jumlah Subscriber. |
| Summary | Ringkasan komik. |
| Update | Hari dalam seminggu untuk update. |
| Reading Link | Tautan di mana komik dapat dibaca. |

### Exploratory Data Analysis
Pada tahap ini, akan dieksplorasi terkait distribusi dan karakteristik data, seperti ringkasan statistik kolom numerik, distribusi Webtoon berdasarkan Genre, Distribusi Webtoon berdasarkan Jadwal Update, dan visualisasi outlier menggunakan Boxplot.

#### Statistik Deskriptif Kolom Likes, Rating, dan Subscribers
| Statistik | Likes        | Rating   | Subscribers   |
|-----------|--------------|----------|----------------|
| Count     | 568.000000   | 568.000000 | 568.000000     |
| Mean      | 3,243,530    | 9.418644   | 482,128.3      |
| Std       | 6,388,215    | 0.557998   | 682,599.1      |
| Min       | 2,434        | 5.410000   | 5,400          |
| 25%       | 233,777.2    | 9.310000   | 114,725        |
| 50%       | 895,588.0    | 9.580000   | 256,000        |
| 75%       | 3,000,000    | 9.730000   | 563,175        |
| Max       | 50,600,000   | 9.930000   | 6,400,000      |

#### Distribusi Genre
![Visualisasi Distribusi Genre](https://github.com/syahelrusfi21/Webtoon-Recommendation-System/blob/main/image/distribusi_genre.png)
Dari plot distribusi `Genre`, terlihat bahwa:

- Genre Fantasy, Romance, dan Drama memiliki jumlah Webtoon (komik) paling banyak berdasarkan sampel data dalam proyek ini. Hal ini 
mengindikasikan bahwa genre tersebut memang banyak ditulis oleh para author. Bisa jadi hal ini karena cerita bergenre fantasy, romance, atau drama lebih laku di pasaran, memiliki peminat yang tinggi, atau jalan ceritanya cenderung lebih ringan ketimbang genre lainnya, sehingga lebih banyak author yang memilih untuk menulis cerita-cerita tersebut.
- Lalu genre seperti historical dan informative, cenderung sedikit jumlah komiknya. Ini menguatkan asumsi saya kalau memang jalan cerita terkait genre ini cenderung lebih sulit dan perlu fokus dan riset mendalam selama penulisannya.

#### Distribusi Update
![Visualisasi Distribusi Update](https://github.com/syahelrusfi21/Webtoon-Recommendation-System/blob/main/image/distribusi_update.png)
Berdasarkan distribusi `Update`, sebagian besar komik Webtoon dari sampel dataset ini merupakan komik yang sudah selesai (*completed*). Selain itu, komik yang masih berlanjut (*ongoing*) rata-rata update per-minggu seperti tiap hari Jumat, Rabu, atau Sabtu. Lalu, ada yang menarik juga bahwa ternyata ada komik yang updatenya cukup sering, yakni setiap hari Senin, Selasa, Rabu, Kamis, dan Minggu, namun jumlahnya hanya sedikit.

## **Data Preparation**
### Penanganan Outlier
Outlier ditangani dengan metode **Winsorization**, di mana nilai ekstrem diubah menjadi batas atas dan bawah berdasarkan IQR (Q1 - 1.5 x IQR dan Q3 + 1.5 x IQR).
- Distribusi data sebelum outlier ditangani
![Distribusi data sebelum outlier ditangani](https://github.com/syahelrusfi21/Webtoon-Recommendation-System/blob/main/image/before_transformation.png)
- Distribusi data setelah outlier ditangani
![Distribusi data setelah outlier ditangani](https://github.com/syahelrusfi21/Webtoon-Recommendation-System/blob/main/image/after_transformation.png)

Setelah proses transformasi, terlihat bahwa outlier telah berhasil dihilangkan. Oleh karena itu, dataset ini sudah cukup bersih untuk digunakan dalam tahap modeling atau analisis lanjutan. Namun, dalam proyek ini, karena akan menerapkan pendekatan **Content-Based Filtering**, dataset asli sebelum transformasi tetap digunakan. Hal ini dikarenakan hanya kolom `Summary`, `Genre`, dan `Name` (judul Webtoon) yang akan dimanfaatkan dalam sistem rekomendasi.

### Penanganan Missing Value
Nilai kosong (*missing values*) pada data hanya ada satu di kolom `Writer`. Oleh karena itu, baris yang memiliki *missing value* tersebut dihapus (*dropping*) untuk memastikan integritas dataset sebelum analisis lebih lanjut.

### Text Preprocessing
  - Teks yang dipreprocessing ialah teks dari kombinasi kolom `Genre` + `Summary`
  - Mengonversi format teks menjadi *lowercase*
  - Menghapus tanda baca dan angka
  - Tokenisasi, menghapus *stopword*, dan *stemming*
  - Ekstraksi fitur menggunakan **TF-IDF Vectorization**

Tujuan dari tahap ini adalah untuk memastikan data berada dalam format yang optimal sebelum dimasukkan ke dalam model rekomendasi.

## Modeling and Results
### Content-Based Filtering
- Menggunakan **cosine similarity** antara ringkasan (*summary*) dan genre dari setiap komik.
- Mengukur kemiripan antar komik berdasarkan representasi tekstual tersebut untuk merekomendasikan komik yang paling mirip dengan komik input.
- Menghasilkan **top-N rekomendasi komik** berdasarkan skor kemiripan tertinggi.

### Contoh Hasil Rekomendasi (Top-5)
Berikut hasil sistem rekomendasi berbasis content-based filtering dengan input judul **"Lookism"**. Rekomendasi didasarkan pada kemiripan konten berupa genre dan ringkasan cerita:

| No | Judul                 | Genre     | Ringkasan                                                                                                                                                                                                                                                                                                      | Keterangan         | Similarity |
|----|-----------------------|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------|------------|
| 1  | Lookism              | Drama     | Daniel is an unattractive loner who wakes up in a different body. Now tall, handsome, and cooler than ever in his new form, Daniel aims to achieve everything he couldn't before. How far will he go to keep his body... and his secrets?                                | Judul yang dicari  | 1.0000     |
| 2  | Fluidum              | Drama     | Jesse was born like every citizen in the Fluidum universe, spending their first 20 years shifting seamlessly between a male and female body. Now with only one year left until they must choose one body for the rest of their life, Jesse is questioning the unquestionable... why decide at all? | Rekomendasi        | 0.1310     |
| 3  | Warning Label        | Romance   | Danielle was cursed by an ex-boyfriend, and now, anytime someone asks her out, they get the warning label. It's scared a few people off, but even the ones who stay wind up bailing when the list turns out to be true.                                                                                        | Rekomendasi        | 0.1307     |
| 4  | Denma                | Sci-fi    | Dike the Invincible Death is the #1 target for bounty hunters across planet Urano. After getting drugged by a mysterious woman in a bar, Dike wakes up in the body of Denma, a 12-year-old space courier. With Denma's powers as a Quanx, he journeys across the galaxy fighting futuristic societies, flying spaceships, and foiling insidious plots, all to get his body back. | Rekomendasi        | 0.1123     |
| 5  | City of Walls        | Drama     | Welcome to the prison-like city of Kowloon. A cluster of conjoined buildings run by gangs. Where the ability of an individual to hope is non-existent. Daniel, Jin, and Ariana are three kids who defy this life sentence. They intend to escape and are willing to go to extreme measures to do so. Since escape on foot is impossible, their quest leads them to the sky. | Rekomendasi        | 0.1028     |
| 6  | My Life as a Loser   | Drama     | My life was ruined after you bullied me in high school. So why do you get to be happy and successful? I'll give you a taste of your own medicine! Now you'll see what it felt like to be me... And you better be ready to pay the price if you want to return to your own body.                                | Rekomendasi        | 0.0932     |

## **Evaluation**
Dalam proyek ini, tidak tersedia data interaksi pengguna untuk melakukan evaluasi kuantitatif menggunakan metrik seperti Precision@K atau Recall@K. Oleh karena itu, evaluasi dilakukan secara **kualitatif**, dengan cara menilai sejauh mana hasil rekomendasi yang dihasilkan oleh sistem terasa relevan berdasarkan isi dan genre Webtoon.

Sistem menggunakan **Cosine Similarity** pada tahap modeling untuk mengukur kemiripan antar Webtoon berdasarkan deskripsi dan genre. Namun perlu dicatat bahwa **Cosine Similarity bukan metrik evaluasi**, melainkan hanya digunakan sebagai dasar untuk menentukan item yang direkomendasikan.

Untuk menilai kualitas hasil, dilakukan analisis terhadap beberapa contoh rekomendasi, misalnya:

- Input: `Lookism`, sebuah Webtoon bertema drama dan kehidupan sekolah.
- Rekomendasi yang muncul seperti `Fluidum`, `City of Walls`, dan `My Life as a Loser`, secara umum masih mengandung elemen-elemen cerita personal, drama, dan konflik sosial, yang membuatnya cukup relevan dari segi konten.

Dengan demikian, meskipun sistem tidak dievaluasi secara kuantitatif, evaluasi kualitatif menunjukkan bahwa pendekatan *content-based* mampu memberikan hasil yang masuk akal dan relevan.


## **Kesimpulan**
Proyek ini berhasil membangun sistem rekomendasi Webtoon menggunakan pendekatan *Content-Based Filtering* dengan *Cosine Similarity* untuk menentukan kemiripan antar judul berdasarkan genre dan ringkasan cerita.

Meskipun nilai kemiripan relatif rendah secara numerik, sistem tetap mampu menghasilkan urutan rekomendasi yang secara kualitatif dinilai relevan. Evaluasi dilakukan secara kualitatif karena tidak tersedia data interaksi pengguna, dan hasilnya menunjukkan bahwa sistem dapat membantu pengguna menemukan Webtoon lain yang serupa dengan minat awalnya.

Pendekatan ini cocok digunakan dalam kondisi terbatas seperti pada dataset ini, dan dapat dikembangkan lebih lanjut dengan *Collaborative Filtering* atau metrik evaluasi kuantitatif jika tersedia data tambahan di masa mendatang.
