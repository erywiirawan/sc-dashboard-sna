# SC KPI Dashboard — Raw Data Templates

**Untuk: PT Saka Niaga (SNA) — Building Material Distributor**
**Dashboard live: https://sc-dashboard-sna.vercel.app**

Template ini berisi 8 CSV file yang jadi "tulang punggung" dashboard. Ery tinggal ganti data sample dengan data real PT SNA, lalu upload ke dashboard.

---

## 📁 Daftar File

| # | File | Tabel | Fungsi |
|---|---|---|---|
| 1 | `01_branches.csv` | Master Cabang | 21 cabang PT SNA + PIC info |
| 2 | `02_suppliers.csv` | Master Supplier | 30 supplier + lead time + TOP |
| 3 | `03_products.csv` | Master SKU | Catalog produk (12 SKU sample, bisa expand) |
| 4 | `04_stock_transactions.csv` | Transaksi Stok | Movement in/out (purchase, sales) |
| 5 | `05_sales_history.csv` | Histori Sales | Bulanan per SKU/cabang + OTIF + Fill Rate |
| 6 | `06_stock_snapshot.csv` | Stok Real-time | Posisi stok + transit + on-order |
| 7 | `07_purchase_orders.csv` | Purchase Order | PO + delivery tracking |
| 8 | `08_customers.csv` | Customer Base | Aktivitas customer per bulan |

---

## 🗂️ Skema Relasional (Sederhana)

```
┌─────────────────┐         ┌──────────────────┐
│ 01_branches     │◄────────│ 05_sales_history │
│ branch_id (PK)  │         │ branch_id (FK)   │
│ region          │         │ sku (FK)         │
│ pic_1/2/3       │         │ sales_qty/value  │
└────────┬────────┘         │ otif_pct         │
         │                  │ fill_rate_pct    │
         │                  └──────────────────┘
         │                          ▲
         │                          │
         │                  ┌───────┴──────────┐
         │                  │ 06_stock_snapshot│
         │                  │ branch_id (FK)   │
         │                  │ sku (FK)         │
         │                  │ stock/transit/   │
         │                  │ on_order         │
         │                  └──────────────────┘
         │
         ▼
┌─────────────────┐         ┌──────────────────┐
│ 02_suppliers    │◄────────│ 07_purchase_orders│
│ supplier_id(PK) │         │ supplier_id (FK) │
│ top_days        │         │ branch_dest (FK) │
│ lead_time_avg   │         │ status           │
└─────────────────┘         │ delivery timing  │
                            └──────────────────┘

┌─────────────────┐         ┌──────────────────┐
│ 03_products     │◄────────│ 04_stock_trans   │
│ sku (PK)        │         │ sku (FK)         │
│ group           │         │ branch_id (FK)   │
│ kategori        │         │ supplier_id (FK) │
│ is_single_source│         │ qty, value       │
└─────────────────┘         └──────────────────┘

┌─────────────────┐
│ 08_customers    │
│ branch_id (FK)  │
│ new/returning   │
│ avg_order_value │
└─────────────────┘
```

---

## 📊 Field Definitions

### 1️⃣ `01_branches.csv` — Master Cabang
| Field | Tipe | Contoh | Keterangan |
|---|---|---|---|
| `branch_id` | string | BWI | Kode unik 3-4 char |
| `branch_name` | string | Saka Niaga Banyuwangi | Nama lengkap |
| `region` | string | BALI / NTB / NTT / KALTIM | Region |
| `address` | string | (alamat lengkap) | Untuk kalkulasi lead time |
| `pic_1_name` | string | Bpk. Hein | PIC utama |
| `pic_1_phone` | string | 0897-xxx | Format +62 atau 08xx |
| `pic_1_role` | string | H (Head) | H=Head, A=Assistant, K=Koord, HO=Head Office |
| `pic_2_*` | ... | ... | PIC backup 1 |
| `pic_3_*` | ... | ... | PIC backup 2 |
| `branch_type` | enum | gudang / transfer | gudang=stok fisik, transfer=stok transit |
| `operational_note` | string | ... | Catatan operasional |
| `opening_date` | date | 2020-01-15 | Tanggal buka cabang |
| `status` | enum | active / inactive | Status operasional |

### 2️⃣ `02_suppliers.csv` — Master Supplier
| Field | Tipe | Contoh | Keterangan |
|---|---|---|---|
| `supplier_id` | int | 1 | Unique ID |
| `supplier_name` | string | Amak Firdaus Utomo | Nama perusahaan |
| `supplier_group` | string | Bata Ringan | Group produk |
| `address` | string | (alamat) | Lokasi supplier |
| `email` | string | (email) | Kontak email |
| `pic_name` | string | (nama PIC) | Sales/PIC supplier |
| `pic_phone` | string | (nomor) | Kontak WA/telp |
| `top_days` | int | 30 | Term of payment (hari), 0=CBD |
| `top_type` | enum | kredit / CBD | kredit=tempo, CBD=cash before delivery |
| `is_primary` | bool | yes/no | Supplier utama untuk group ini? |
| `is_backup` | bool | yes/no | Backup supplier? |
| `lead_time_days_min/max/avg/std` | int | 14/21/17/3 | Lead time (hari) + std dev |
| `quality_rating` | float | 4.5 | Skor 1-5 (1=jelek, 5=bagus) |
| `otif_rating` | int | 90 | % OTIF supplier (0-100) |
| `contract_start/end` | date | 2023-01-01 | Masa kontrak |
| `payment_method` | enum | transfer / cash | Cara bayar ke supplier |

### 3️⃣ `03_products.csv` — Master SKU
| Field | Tipe | Contoh | Keterangan |
|---|---|---|---|
| `sku` | string | 05SGMB914 | SKU code (unique) |
| `sku_name` | string | SENG GAJAMADA 0.20 | Nama lengkap |
| `group` | string | Seng | Group produk (link ke supplier) |
| `kategori` | enum | structural / infrastructure / utilities | Kategori |
| `unit_of_measure` | string | lembar / kg / m3 / roll | Unit jual |
| `package_size` | int | 1 | Isi per package |
| `weight_kg` | float | 2.5 | Berat (untuk hitung ongkir) |
| `length/width/height_cm` | float | ... | Dimensi (opsional) |
| `hs_code` | string | 72104110 | HS code (untuk bea cukai) |
| `is_active` | bool | yes/no | Masih dijual? |
| `is_critical` | bool | yes/no | SKU critical (impact tinggi) |
| `is_single_source` | bool | yes/no | Single-source supplier? |

### 4️⃣ `04_stock_transactions.csv` — Transaksi Stok
| Field | Tipe | Contoh | Keterangan |
|---|---|---|---|
| `txn_id` | string | TXN001 | Unique ID |
| `date` | date | 2026-01-05 | Tanggal transaksi |
| `branch_id` | FK | BWI | Cabang asal/tujuan |
| `sku` | FK | 05SGMB914 | SKU |
| `txn_type` | enum | in / out / transfer / adjustment | Tipe movement |
| `quantity` | int | 50 | Jumlah |
| `unit` | string | lembar | Unit |
| `unit_price_idr` | int | 47500 | Harga per unit |
| `total_value_idr` | int | 2375000 | Total nilai (qty × price) |
| `supplier_id` | FK | 1 | Supplier (untuk in) |
| `po_number` | string | PO-2026-001 | Link ke PO (untuk in) |
| `notes` | string | ... | Catatan |

### 5️⃣ `05_sales_history.csv` — Histori Sales (Bulanan)
**Ini yang paling penting untuk dashboard KPI!**
| Field | Tipe | Contoh | Keterangan |
|---|---|---|---|
| `period_date` | date | 2026-01-31 | Akhir bulan |
| `branch_id` | FK | BWI | Cabang |
| `sku` | FK | 05SGMB914 | SKU |
| `sales_qty` | int | 55 | Quantity terjual |
| `sales_value_idr` | int | 2860000 | Nilai penjualan |
| `cogs_idr` | int | 2227500 | Cost of goods sold |
| `gross_margin_idr` | int | 632500 | Margin (sales - COGS) |
| `order_count` | int | 12 | Jumlah order masuk bulan ini |
| `on_time_count` | int | 10 | Order dikirim tepat waktu |
| `in_full_count` | int | 11 | Order dikirim lengkap |
| `otif_pct` | float | 83.3 | OTIF % (auto-calc: on_time / order_count × 100) |
| `fill_rate_pct` | float | 91.7 | Fill rate % (in_full / order_count × 100) |

### 6️⃣ `06_stock_snapshot.csv` — Posisi Stok (Real-time)
| Field | Tipe | Contoh | Keterangan |
|---|---|---|---|
| `snapshot_date` | date | 2026-06-01 | Tanggal snapshot |
| `branch_id` | FK | BWI | Cabang |
| `sku` | FK | 05SGMB914 | SKU |
| `stock_qty` | int | 0 | Stok di tangan |
| `transit_qty` | int | 0 | Dalam perjalanan ke cabang |
| `on_order_qty` | int | 100 | Sudah dipesan (belum dikirim) |
| `committed_qty` | int | 30 | Sudah dipesan customer (reserved) |
| `available_qty` | int | 0 | Tersedia (stock - committed) |
| `reserved_qty` | int | 0 | Reserved (sama dengan committed) |
| `days_of_cover` | int | 0 | Berapa hari stok cukup |
| `reorder_point` | int | 50 | Batas minimum stok |
| `safety_stock` | int | 30 | Stok pengaman |
| `status` | enum | urgent_stockout / low_stock / normal / overstock | Status |
| `last_count_date` | date | 2026-05-31 | Terakhir stock opname |

### 7️⃣ `07_purchase_orders.csv` — Purchase Order Tracking
| Field | Tipe | Contoh | Keterangan |
|---|---|---|---|
| `po_id` | string | PO-2026-001 | PO number |
| `po_date` | date | 2026-01-05 | Tanggal PO dibuat |
| `supplier_id` | FK | 1 | Supplier |
| `branch_dest` | FK | BWI | Cabang tujuan |
| `expected_date` | date | 2026-01-26 | Expected arrival |
| `actual_delivery_date` | date | 2026-01-27 | Actual arrival |
| `status` | enum | scheduled / in_transit / completed / cancelled | Status |
| `total_value_idr` | int | 2375000 | Nilai PO |
| `total_items` | int | 1 | Jumlah line item |
| `delivery_on_time` | bool | yes/no/na | Tepat waktu? |
| `delivery_in_full` | bool | yes/no/na | Lengkap? |
| `quality_accepted` | bool | yes/no/na | Quality OK? |
| `quality_rejected` | bool | yes/no/na | Ada reject? |
| `notes` | string | ... | Catatan |

### 8️⃣ `08_customers.csv` — Customer Analytics
| Field | Tipe | Contoh | Keterangan |
|---|---|---|---|
| `period_date` | date | 2026-01-31 | Akhir bulan |
| `branch_id` | FK | BWI | Cabang |
| `new_customers` | int | 5 | Customer baru bulan ini |
| `returning_customers` | int | 42 | Customer repeat order |
| `total_active_customers` | int | 180 | Total customer aktif |
| `total_orders` | int | 225 | Order masuk |
| `total_visits` | int | 520 | Kunjungan sales |
| `avg_order_value_idr` | int | 4850000 | Rata-rata nilai order |
| `top_customer_segment` | enum | kontraktor / proyek_villa / toko_bangunan / proyek_pemerintah | Segmen dominan |

---

## 🛠️ Cara Pakai

### Opsi A: Edit Manual di Excel/Google Sheets
1. Download CSV dari folder `/root/sc-dashboard-sna/templates/`
2. Buka di Excel / Google Sheets
3. Replace data sample dengan data real PT SNA
4. Save as `.csv` (UTF-8, comma-separated)
5. Upload ke dashboard (perlu adaptor — saat ini dashboard baca dari hardcoded JS, **akan saya update supaya bisa load CSV**)

### Opsi B: Generate dari Sistem Existing PT SNA
Kalau PT SNA sudah pakai ERP/software:
1. Export master data ke CSV sesuai schema di atas
2. Upload ke dashboard

### Opsi C: Sync Langsung (untuk advanced)
1. Setup Google Sheets / Airtable sebagai master data
2. Dashboard auto-pull via API (akan saya implement jika Ery mau)

---

## 📌 Quick Start

**Untuk langsung pakai, Ery cukup:**
1. **Edit `05_sales_history.csv`** — ini data utama untuk KPI (OTIF, Fill Rate, Sales Trend)
2. **Edit `06_stock_snapshot.csv`** — untuk inventory turnover, days on hand
3. **Edit `02_suppliers.csv`** — lead time & TOP untuk supplier scorecard

**File lain (branches, products, customers) optional** — sudah ada data sample yang representatif.

---

## 📊 KPI yang Bisa Dihitung dari Data Ini

| KPI | Sumber Data | Formula |
|---|---|---|
| **OTIF** | sales_history | AVG(otif_pct) per branch |
| **Fill Rate** | sales_history | AVG(fill_rate_pct) per branch |
| **Inventory Turnover** | sales_history + stock_snapshot | COGS / avg(stock_value) |
| **Days on Hand** | stock_snapshot | AVG(days_of_cover) |
| **Stockout Rate** | stock_snapshot | COUNT(stock=0) / total |
| **Working Capital** | stock_snapshot | SUM(stock × cost) |
| **Supplier OTIF** | purchase_orders | AVG(on_time / total) |
| **Lead Time Variance** | purchase_orders | STD(actual - expected) |
| **Customer Retention** | customers | returning / total_active |
| **Average Order Value** | customers | AVG(order_value) |

---

## ❓ Next Step

Pilih salah satu:

1. **Update dashboard** supaya bisa load CSV dari URL (Ery bisa edit di Google Sheets, dashboard auto-refresh)
2. **Buat Google Sheets template** dengan formula otomatis (siap diisi)
3. **Generate sample data lebih banyak** (misal 67 SKU × 21 cabang × 5 bulan)
4. **Konsultasi cara mapping** dari data existing PT SNA ke schema ini

Atau kalau Ery mau langsung **isi data real** sekarang juga, saya bisa bantu mapping field-by-field.
