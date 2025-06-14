# 📦 Data Warehouse Ekspor-Impor Komoditas Pertanian

Proyek ini bertujuan untuk membangun sistem Data Warehouse untuk perusahaan ekspor-impor sektor agribisnis (contoh: kopi, teh, kelapa sawit), dan melakukan analisis berbasis data terhadap aktivitas ekspor secara end-to-end menggunakan Python, MySQL, SQLAlchemy, dan visualisasi di Jupyter Notebook.

---

## 🧱 Struktur Data Warehouse

### 📊 Tabel Dimensi
- `dim_product`: informasi produk agribisnis (nama, kategori, kualitas).
- `dim_country`: negara tujuan ekspor dan regionalnya.
- `dim_date`: kalender transaksi.
- `dim_port`: informasi pelabuhan pengiriman.
- `dim_customer`: pelanggan luar negeri (importir).
- `dim_shipping_agent`: agen pengiriman dan jenis layanannya.

### 📈 Tabel Fakta
- `fact_exports`: mencatat seluruh transaksi ekspor (produk, negara, tanggal, customer, port, shipping agent) serta nilai ekspor, ongkos kirim, bea cukai, dan asuransi.

---

## ⚙️ Koneksi Database ke Jupyter Notebook

Menggunakan SQLAlchemy:

```python
from sqlalchemy import create_engine

user = "root"
password = "ini saya rahasiakan yaa"
host = "localhost"
port = 3306
database = "export_import_dw"

connection_string = f"mysql+pymysql://{user}:{password}@{host}:{port}/{database}"
engine = create_engine(connection_string)

print("✅ Koneksi berhasil disiapkan. Engine siap digunakan.")

```
---

## 🗃️ Pembuatan Tabel dengan Magic SQL (%%sql)
Semua tabel dimensi dan fakta dibuat langsung di Jupyter Notebook dengan sintaks SQL menggunakan cell ```%%sql```.

```sql
%%sql
CREATE TABLE dim_product (
    product_id INT AUTO_INCREMENT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    grade VARCHAR(20),
    unit VARCHAR(20)
);
```
Ulangi untuk semua tabel lainnya (dim_country, dim_date, dst).

---
## 🧪 Insert Data Dummy
Data dummy sebanyak 10 transaksi ekspor dimasukkan langsung ke semua tabel. Nilai export_value dihitung dari ```quantity * unit_price```.

---

## 🔍 Validasi Data
Untuk memastikan semua tabel berhasil dibuat dan terisi:
```python
with engine.connect() as conn:
    result = conn.execute(text("SHOW TABLES"))
    for row in result:
        print(row[0])
```
Dan untuk melihat isi tabel:
```sql
%%sql
SELECT * FROM fact_exports;
```
---

## 🔄 Simulasi ETL
### Extract
Data dari seluruh tabel di-join dengan ```fact_exports``` untuk menghasilkan tabel lengkap.
```sql
%%sql
SELECT 
    f.export_id, f.quantity, f.unit_price, f.export_value,
    f.shipping_cost, f.customs_tax, f.insurance_cost,
    c.country_code, c.country_name, c.region,
    d.date_id, d.year, d.month
FROM fact_exports f
JOIN dim_country c ON f.country_code = c.country_code
JOIN dim_date d ON f.date_id = d.date_id;
```

### Transform
Data dibersihkan dan di-aggregate berdasarkan tahun, bulan, dan negara:
```python
df_aggr = df_fact.groupby(
    ['year', 'month', 'country_name', 'region']
).agg({
    'export_value': 'sum',
    'quantity': 'sum',
    'shipping_cost': 'sum',
    'customs_tax': 'sum',
    'insurance_cost': 'sum'
}).reset_index()
```

### Load
Data hasil transform dimasukkan ke tabel agregat baru di database:
```python
df_aggr.to_sql("agg_export_summary", con=engine, if_exists='replace', index=False)
```
---

## 📊 Analisis & Visualisasi
### 📈 Time Series Ekspor
Menganalisis tren total ekspor dari bulan ke bulan:
```python
df_timeseries = df_aggr.groupby(['year', 'month'])['export_value'].sum().reset_index()
```
> Visualisasi: Garis waktu menggunakan ```matplotlib```.

### 💰 Profitability per Negara/Produk
Perhitungan margin keuntungan ekspor:
```python
df_profit = df_fact.copy()
df_profit['profit'] = (
    df_profit['export_value'] 
    - df_profit['shipping_cost'] 
    - df_profit['customs_tax'] 
    - df_profit['insurance_cost']
)
```
> Di-groupby berdasarkan ```country_name``` atau ```product_name```.

## 🚢 Shipping Analysis
Analisis efisiensi biaya pengiriman per negara atau per shipping agent:
```python
shipping_eff = df_fact.groupby("country_name").agg({
    "shipping_cost": "mean",
    "export_value": "sum"
}).reset_index()
```
---

## 📌 Tools & Library
- Python + Jupyter Notebook
- MySQL 8.x
- SQLAlchemy
- Pandas, Matplotlib, Seaborn
- ipython-sql magic (%load_ext sql)
---

## 🏁 Kesimpulan
Proyek ini berhasil membangun dan mensimulasikan Data Warehouse untuk sektor ekspor-impor agribisnis secara end-to-end:
- Desain skema multidimensi (star schema)
- Insert dan ETL data realistis
- Analisis profitabilitas dan tren ekspor
- Visualisasi mendalam
---

## 📂 Struktur Proyek
```
├── notebooks/
│   └── export_import_analysis.ipynb
├── README.md
```
---

## ✍️ Author:
> Danang Hilal Kurniawan
