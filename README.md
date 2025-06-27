
# Analisis Data Cuaca Seattle Menggunakan Apache Spark

## Langkah 1: Menjalankan Hadoop dan Spark

Sebelum menjalankan analisis, pastikan Hadoop dan Spark berhasil dijalankan secara lokal.

### 🔹 Jalankan HDFS dan Spark

```bash
start-dfs.cmd
start-yarn.cmd
spark-shell
```

---

## Langkah 2: Membuat Struktur Folder Proyek

Agar proyek rapi dan terorganisir, buat struktur folder berikut:

### 🔹 Perintah CMD untuk Membuat Folder

```cmd
cd %USERPROFILE%\Documents && mkdir TUGAS_BIG_DATA && cd TUGAS_BIG_DATA && mkdir data && mkdir analisis && mkdir analisis_visualisasi

start .  // untuk membuka lokasi folder di File Explorer
```

---

## Langkah 3: Memasukkan Dataset

1. Cari file `seattle-weather.csv`.
2. Pindahkan ke folder `TUGAS_BIG_DATA/data/`.
3. Ketik `code .` di CMD untuk membuka Visual Studio Code.

---

## Langkah 4: Membuat dan Menjalankan File `analisis_cuaca.py`

### 📄 Buat file `analisis/analisis_cuaca.py`

### ✏️ Isi dengan kode berikut:

```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import col, avg, when, month, year, max as max_, concat_ws

spark = SparkSession.builder.appName("AnalisisCuacaSeattle").getOrCreate()
spark.sparkContext.setLogLevel("ERROR")

df = spark.read.option("header", "true").csv("../data/seattle-weather.csv", inferSchema=True)

# 1. Rata-rata Suhu Harian
df_avg = df.select("date", "temp_max", "temp_min")            .withColumn("avg_temp", (col("temp_max") + col("temp_min")) / 2)
df_avg.select(col("date").alias("tanggal"), col("avg_temp").alias("suhu_rata_rata"))       .write.csv("../output/avg_temp_harian", header=True, mode="overwrite")

# 2. Hari Suhu Ekstrem
hari_max = df.orderBy(df.temp_max.desc()).first()
hari_min = df.orderBy(df.temp_min.asc()).first()
hasil_ekstrem = spark.createDataFrame([
    (f"Hari dengan suhu maksimum tertinggi: {hari_max['date']} ({hari_max['temp_max']} °C)",),
    (f"Hari dengan suhu minimum terendah: {hari_min['date']} ({hari_min['temp_min']} °C)",)
], ["info_suhu_ekstrem"])
hasil_ekstrem.write.csv("../output/hari_suhu_ekstrem", header=True, mode="overwrite")

# 3. Suhu Maksimum Bulanan
df_bulan = df.withColumn("bulan", concat_ws("-", year("date"), month("date")))
df_bulan.groupBy("bulan").agg(max_("temp_max").alias("suhu_maksimum"))         .orderBy("bulan")         .write.csv("../output/suhu_maksimum_bulanan", header=True, mode="overwrite")

# 4. Rata-rata Angin per Cuaca
df_cuaca = df.withColumn("cuaca", when(col("weather") == "rain", "hujan")
                         .when(col("weather") == "sun", "cerah")
                         .when(col("weather") == "snow", "salju")
                         .when(col("weather") == "fog", "berkabut")
                         .when(col("weather") == "drizzle", "gerimis")
                         .otherwise(col("weather")))
df_cuaca.groupBy("cuaca").agg(avg("wind").alias("rata_rata_kecepatan_angin"))        .write.csv("../output/rata_rata_angin_per_cuaca", header=True, mode="overwrite")

# 5. Jumlah Kondisi Cuaca
df_kondisi = df.withColumn("kondisi_cuaca", when(col("weather") == "rain", "Hujan")
                           .when(col("weather") == "sun", "Cerah")
                           .when(col("weather") == "snow", "Salju")
                           .when(col("weather") == "drizzle", "Gerimis")
                           .when(col("weather") == "fog", "Berkabut")
                           .otherwise(col("weather")))
df_kondisi.groupBy("kondisi_cuaca").count().orderBy("count", ascending=False)           .write.csv("../output/jumlah_kondisi_cuaca", header=True, mode="overwrite")

spark.stop()
```

### ▶️ Jalankan File

```bash
cd analisis
spark-submit analisis_cuaca.py
```

---

## Langkah 5: Hasil Output Analisis

Folder `output/` akan otomatis terisi dengan:

```
output/
├── avg_temp_harian/
├── hari_suhu_ekstrem/
├── suhu_maksimum_bulanan/
├── rata_rata_angin_per_cuaca/
└── jumlah_kondisi_cuaca/
```

---

## Langkah 6: Visualisasi Data Hasil Analisis

### ✅ Install Library

```bash
pip install pandas matplotlib seaborn
```

### 📁 Buat folder `visualisasi/` (otomatis dibuat oleh setiap script)

---

## Langkah 7: Visualisasi Hasil Analisis

### A. Rata-Rata Suhu Harian

**File:** `Visualisasi_rata_rata_suhu.py`  
**Output:** `visualisasi/rata_rata_suhu_harian.png`

### B. Hari Suhu Ekstrem

**File:** `visualisasi_suhu_ekstrem.py`  
**Output:** `visualisasi/suhu_ekstrem.png`

### C. Suhu Maksimum Bulanan

**File:** `visualisasi_suhu_maksimum_per_bulan.py`  
**Output:** `visualisasi/suhu_maksimum_per_bulan.png`

### D. Rata-rata Kecepatan Angin per Cuaca

**File:** `visualisasi_rata_rata_angin.py`  
**Output:** `visualisasi/rata_rata_angin_per_cuaca.png`

### E. Jumlah Kondisi Cuaca

**File:** `visualisasi_jumlah_hari_hujan.py`  
**Output:** `visualisasi/frekuensi_kondisi_cuaca.png`
