
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

### 📦 Pastikan Library Visualisasi Terinstal

```bash
pip install pandas matplotlib seaborn
```

---

### A. Rata-Rata Suhu Harian

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import glob
import os

os.makedirs("visualisasi", exist_ok=True)
sns.set(style="whitegrid")

avg_temp_files = glob.glob("output/avg_temp_harian/part-*.csv")
df_avg_temp = pd.concat([pd.read_csv(f, names=["Tanggal", "Suhu Rata-rata"], header=0) for f in avg_temp_files])
df_avg_temp["Tanggal"] = pd.to_datetime(df_avg_temp["Tanggal"])

plt.figure(figsize=(14, 5))
plt.plot(df_avg_temp["Tanggal"], df_avg_temp["Suhu Rata-rata"], color='orange')
plt.title("Rata-rata Suhu Harian di Seattle")
plt.xlabel("Tanggal")
plt.ylabel("Suhu Rata-rata (C)")
plt.tight_layout()
plt.savefig("visualisasi/rata_rata_suhu_harian.png")
plt.close()
```

---

### B. Hari Suhu Ekstrem

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import glob
import os
import re

os.makedirs("visualisasi", exist_ok=True)
sns.set(style="whitegrid")

extreme_files = glob.glob("output/hari_suhu_ekstrem/part-*.csv")
if extreme_files:
    df_extreme = pd.concat([pd.read_csv(f, header=None, names=["Keterangan"]) for f in extreme_files])
    pattern_max = r"maksimum tertinggi: (\d{4}-\d{2}-\d{2}).*\(([-+]?\d+\.\d+)"
    pattern_min = r"minimum terendah: (\d{4}-\d{2}-\d{2}).*\(([-+]?\d+\.\d+)"
    max_match = df_extreme["Keterangan"].str.extract(pattern_max).dropna()
    min_match = df_extreme["Keterangan"].str.extract(pattern_min).dropna()

    if not max_match.empty and not min_match.empty:
        data = pd.DataFrame({
            "Jenis": ["Suhu Maksimum Tertinggi", "Suhu Minimum Terendah"],
            "Tanggal": [max_match.iloc[0, 0], min_match.iloc[0, 0]],
            "Suhu": [float(max_match.iloc[0, 1]), float(min_match.iloc[0, 1])]
        })

        plt.figure(figsize=(8, 5))
        sns.barplot(data=data, x="Suhu", y="Jenis", palette="coolwarm")
        for i in range(len(data)):
            plt.text(data["Suhu"][i], i, f'{data["Suhu"][i]} °C', va='center', ha='left')
        plt.title("Hari dengan Suhu Ekstrem di Seattle")
        plt.xlabel("Suhu (°C)")
        plt.ylabel("")
        plt.tight_layout()
        plt.savefig("visualisasi/suhu_ekstrem.png")
        plt.close()
```

---

### C. Suhu Maksimum Bulanan

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import glob
import os

os.makedirs("visualisasi", exist_ok=True)
sns.set(style="whitegrid")

monthly_max_files = glob.glob("output/suhu_maksimum_bulanan/part-*.csv")
df_monthly_max = pd.concat([
    pd.read_csv(f, header=None, names=["bulan", "suhu_maks"])
    for f in monthly_max_files
])

plt.figure(figsize=(14, 5))
sns.barplot(data=df_monthly_max, x="bulan", y="suhu_maks", color='skyblue')
plt.xticks(rotation=45)
plt.title("Suhu Maksimum per Bulan")
plt.xlabel("Bulan")
plt.ylabel("Suhu Maksimum (°C)")
plt.tight_layout()
plt.savefig("visualisasi/suhu_maksimum_per_bulan.png")
plt.close()
```

---

### D. Rata-Rata Angin per Jenis Cuaca

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import glob
import os

os.makedirs("visualisasi", exist_ok=True)
sns.set(style="whitegrid")

wind_files = glob.glob("output/rata_rata_angin_per_cuaca/part-*.csv")
if wind_files:
    df_wind = pd.concat([
        pd.read_csv(f, header=None, names=["cuaca", "rata_rata_angin"])
        for f in wind_files
    ])

    plt.figure(figsize=(10, 5))
    sns.barplot(data=df_wind, x="cuaca", y="rata_rata_angin", palette="Blues_d")
    plt.title("Rata-rata Kecepatan Angin per Jenis Cuaca")
    plt.xlabel("Jenis Cuaca")
    plt.ylabel("Rata-rata Kecepatan Angin (m/s)")
    plt.tight_layout()
    plt.savefig("visualisasi/rata_rata_angin_per_cuaca.png")
    plt.close()
```

---

### E. Frekuensi Kondisi Cuaca

```python
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import glob
import os

os.makedirs("visualisasi", exist_ok=True)
sns.set(style="whitegrid")

weather_count_files = glob.glob("output/jumlah_kondisi_cuaca/part-*.csv")
if weather_count_files:
    df_weather = pd.concat([
        pd.read_csv(f, header=None, names=["kondisi_cuaca", "count"])
        for f in weather_count_files
    ])

    plt.figure(figsize=(10, 5))
    sns.barplot(data=df_weather, x="kondisi_cuaca", y="count", palette="Set2")
    plt.title("Frekuensi Masing-masing Kondisi Cuaca")
    plt.xlabel("Jenis Cuaca")
    plt.ylabel("Jumlah Hari")
    plt.tight_layout()
    plt.savefig("visualisasi/frekuensi_kondisi_cuaca.png")
    plt.close()
```
