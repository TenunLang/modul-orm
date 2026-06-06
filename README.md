# modul-orm

ORM untuk bahasa [Tenun](https://github.com/TenunLang/Tenun): satu API CRUD untuk **MySQL** dan **PostgreSQL**. Bergantung pada `modul-mysql` dan `modul-postgres` (otomatis terpasang).

## Pasang

```
tenun add orm
```

(`tenun add` otomatis memasang dependensi: `mysql` dan `postgres`.)

## Pakai

```tenun
impor "orm";

biar db: bulat = orm_sambung("mysql", "127.0.0.1", 3306, "root", "", "tokoku");
// atau: orm_sambung("postgres", "localhost", 5432, "postgres", "rahasia", "tokoku")

// CREATE (INSERT)
orm_sisip(db, "pengguna", ["nama", "kota"], ["Budi", "Jakarta"]);

// READ
biar h: teks = orm_cari(db, "pengguna", "kota", "Jakarta");
biar i: bulat = 0;
selama i < orm_baris(h) {
    cetak(orm_ambil(h, i, 0));   // kolom ke-0
    i = i + 1;
}

// UPDATE
orm_ubah(db, "pengguna", ["kota"], ["Bandung"], "nama", "Budi");

// DELETE
orm_hapus(db, "pengguna", "nama", "Budi");

orm_tutup(db);
```

## Fungsi

Koneksi:

- `orm_sambung(driver, inang, port, pengguna, sandi, db): bulat` — `driver` = `"mysql"` | `"postgres"`. Kembalikan handle, atau `-1`.
- `orm_tutup(soket): kosong`

CRUD:

- `orm_semua(soket, tabel): teks` — `SELECT * FROM tabel`.
- `orm_cari(soket, tabel, kolom, nilai): teks` — `SELECT * ... WHERE kolom = nilai`.
- `orm_sisip(soket, tabel, kolom: []teks, nilai: []teks): teks` — INSERT.
- `orm_ubah(soket, tabel, setKolom: []teks, setNilai: []teks, whereKolom, whereNilai): teks` — UPDATE.
- `orm_hapus(soket, tabel, kolom, nilai): teks` — DELETE.

Skema / migrasi:

- `orm_buat_tabel(soket, tabel, kolom: []teks, tipe: []teks): teks` — CREATE TABLE IF NOT EXISTS.
- `orm_hapus_tabel(soket, tabel): teks` — DROP TABLE IF EXISTS.
- `orm_tambah_kolom(soket, tabel, kolom, tipe): teks` — ALTER TABLE ADD COLUMN.

Query lanjutan:

- `orm_hitung(soket, tabel): bulat` — COUNT(*).
- `orm_cari_satu(soket, tabel, kolom, nilai): teks` — WHERE + LIMIT 1.
- `orm_urut(soket, tabel, kolom, arah, batas): teks` — ORDER BY + LIMIT (`arah` = "ASC"/"DESC").

Relasi:

- `orm_punya_banyak(soket, tabelAnak, fk, nilaiInduk): teks` — hasMany (anak yang `fk = nilaiInduk`).
- `orm_milik(soket, tabelInduk, pk, nilaiFk): teks` — belongsTo (induk dengan `pk = nilaiFk`, LIMIT 1).
- `orm_gabung(soket, kiri, kanan, kunciKiri, kunciKanan): teks` — INNER JOIN.
- `orm_gabung_cari(soket, kiri, kanan, kunciKiri, kunciKanan, whereKolom, whereNilai): teks` — JOIN + filter berparameter.

Query builder berantai (chainable, ala Eloquent):

```tenun
biar b: peta = qb("pengguna");
b = qb_pilih(b, "id, nama, umur");
b = qb_dimana(b, "umur", ">=", "18");
b = qb_seperti(b, "nama", "%budi%");
b = qb_dimana_dalam(b, "kota", ["Bandung", "Jakarta"]);
b = qb_antara(b, "skor", "10", "90");
b = qb_urut(b, "umur", "DESC");
b = qb_halaman(b, 2, 10);            // halaman 2, 10/halaman
biar hasil: teks = qb_ambil(db, b);  // SELECT berparameter, aman injeksi
cetak(qb_hitung(db, b));             // COUNT(*) dengan filter sama
cetak(qb_agregat(db, b, "AVG", "umur"));
```

- `qb(tabel)` · `qb_pilih` · `qb_dimana(kolom, op, nilai)` · `qb_atau` · `qb_seperti` · `qb_dimana_dalam` · `qb_antara` · `qb_kosong` · `qb_tidak_kosong` · `qb_urut` · `qb_kelompok` · `qb_batas` · `qb_lewati` · `qb_halaman`.
- Eksekusi: `qb_ambil`, `qb_pertama`, `qb_hitung`, `qb_agregat(s, b, "SUM"|"AVG"|"MIN"|"MAX", kolom)`, `qb_hapus`. Debug: `qb_sql(b)`.

Transaksi & aksi tulis lanjutan:

- `orm_transaksi(soket)` / `orm_komit(soket)` / `orm_batal(soket)` — BEGIN/COMMIT/ROLLBACK.
- `orm_id_terakhir(soket): bulat` — id auto-increment terakhir (LAST_INSERT_ID / LASTVAL).
- `orm_sisip_id(soket, tabel, kolom, nilai): bulat` — INSERT lalu kembalikan id baru.
- `orm_sisip_banyak(soket, tabel, kolom, baris: [][]teks): teks` — bulk INSERT satu pernyataan.
- `orm_upsert(soket, tabel, kolom, nilai, konflik): teks` — INSERT atau UPDATE bila bentrok (driver-aware: ON DUPLICATE KEY / ON CONFLICT).

Skema lanjutan (driver-aware):

- `orm_tipe_id(): teks` — tipe id auto-increment PK (`INT AUTO_INCREMENT PRIMARY KEY` / `SERIAL PRIMARY KEY`).
- `orm_indeks(soket, tabel, kolom)` / `orm_indeks_unik(...)` — CREATE [UNIQUE] INDEX.
- `orm_kunci_asing(soket, tabel, fk, induk, pk)` — ADD FOREIGN KEY.

Relasi banyak-ke-banyak (pivot):

- `orm_milik_banyak(soket, tabelTujuan, pivot, fkLokal, fkTujuan, pkTujuan, nilaiLokal): teks` — belongsToMany.
- `orm_tautkan(...)` / `orm_lepas(...)` — attach/detach pasangan pivot.

Query berparameter & baca hasil:

- `orm_jalan(soket, sql, params: []teks): teks` — jalankan SQL dengan placeholder `?` (nilai dikirim terpisah, aman injeksi).
- `orm_kueri(soket, sql): teks` — query tanpa parameter. **Hanya untuk SQL tepercaya (DDL, `SELECT *`).** Jangan sisipkan nilai pengguna.
- `orm_baris(hasil): bulat` — jumlah baris.
- `orm_ambil(hasil, baris, kolom): teks` — nilai sel (0-indeks).

## Keamanan (anti SQL injection)

Nilai TIDAK pernah digabung mentah ke SQL:

- **PostgreSQL** memakai prepared statement (extended protocol, `$1..`) — nilai dikirim terpisah dari perintah.
- **MySQL** memakai escape kuat (`\ ' " \n \r NUL Ctrl-Z`) lalu dikutip.

Semua fungsi CRUD memakai jalur berparameter ini. Contoh injeksi `x' OR '1'='1` diperlakukan sebagai literal (0 baris), bukan SQL.

Nama tabel/kolom diambil dari kode, bukan input pengguna.

## Struktur

```
modul-orm/
  tenun.json            (butuh: mysql, postgres)
  src/
    orm.tenun           entry — impor mysql, postgres, dan berkas di bawah
    inti/
      koneksi.tenun     orm_sambung / orm_tutup
      kueri.tenun       orm_jalan / orm_kueri / orm_baris / orm_ambil
    crud/
      crud.tenun        orm_semua / orm_cari / orm_sisip / orm_ubah / orm_hapus
      skema.tenun       buat/hapus tabel, kolom, indeks, FK, orm_tipe_id
      query.tenun       orm_hitung / orm_cari_satu / orm_urut
      transaksi.tenun   orm_transaksi / orm_komit / orm_batal / orm_id_terakhir
      tindakan.tenun    orm_sisip_banyak / orm_upsert / orm_sisip_id
    builder/
      builder.tenun     query builder berantai (qb_*)
    relasi/
      relasi.tenun      hasMany / belongsTo / JOIN / belongsToMany (pivot)
    model/
      model.tenun       model (peta) + soft-delete + findOrCreate/updateOrCreate
```

Impor antar-berkas relatif terhadap berkas pengimpor (mendukung subfolder, mis. `impor "./inti/koneksi.tenun";`).

## Definisi model (ala Sequelize)

Model dibangun dari tipe `peta` bawaan Tenun: tabel + kolom + tipe SQL. Lihat `contoh/model.tenun`.

```tenun
biar Pengguna: peta = orm_model("pengguna");
Pengguna = orm_kolom(Pengguna, "id", "INT PRIMARY KEY AUTO_INCREMENT");
Pengguna = orm_kolom(Pengguna, "nama", "VARCHAR(100)");
Pengguna = orm_kolom(Pengguna, "umur", "INT");

orm_migrasi(db, Pengguna);                                   // CREATE TABLE IF NOT EXISTS
orm_simpan(db, Pengguna, peta{ "nama": "Taqin", "umur": "27" });  // INSERT berparameter

biar hasil: teks = orm_model_semua(db, Pengguna);
untuk i dari 0 sampai orm_baris(hasil) {
    biar baris: peta = orm_baris_peta(Pengguna, hasil, i);   // baris -> peta kolom:nilai
    cetak(baris["nama"] + " (" + baris["umur"] + ")");
}
```

### Sinkronisasi, query lanjutan, validasi (ala Sequelize)

```tenun
biar U: peta = orm_model("pengguna");
U = orm_kolom(U, "id", "INT PRIMARY KEY AUTO_INCREMENT");
U = orm_kolom(U, "nama", "VARCHAR(100)");
U = orm_timestamps(U);          // tambah dibuat_pada/diubah_pada (default CURRENT_TIMESTAMP)
U = orm_wajib(U, "nama");       // tandai kolom wajib

orm_sinkron(db, U);             // CREATE IF NOT EXISTS (sequelize sync)
// orm_sinkron_paksa(db, U);    // DROP lalu CREATE (sync force:true) — hapus data

kalau orm_validasi(U, peta{ "nama": "Sri" }) {
    orm_simpan(db, U, peta{ "nama": "Sri", "umur": "33" });
}

biar p: peta = orm_model_cari_satu(db, U, "nama", "Sri");   // findOne -> peta
cetak(orm_model_hitung(db, U));                              // count(*)
orm_model_ubah(db, U, peta{ "umur": "34" }, "nama", "Sri"); // UPDATE
cetak(orm_model_ada(db, U, "nama", "Budi"));                 // exists?
```

Fungsi model: `orm_model`, `orm_kolom`, `orm_model_kolom`, `orm_timestamps`, `orm_wajib`, `orm_validasi`, `orm_sinkron`, `orm_sinkron_paksa`, `orm_migrasi`, `orm_model_hapus_tabel`, `orm_simpan`, `orm_model_semua`, `orm_model_cari`, `orm_model_cari_satu`, `orm_model_hitung`, `orm_model_ada`, `orm_model_ubah`, `orm_model_hapus`, `orm_model_kosongkan`, `orm_baris_peta`. Semua nilai berparameter (aman injeksi); diuji live di MySQL/MariaDB & PostgreSQL (termasuk kolom TIMESTAMP).

### Soft delete & findOrCreate (model)

```tenun
U = orm_pakai_softdelete(U);                 // tambah kolom dihapus_pada
orm_model_hapus_lunak(db, U, "id", "7");     // tandai terhapus (baris tetap ada)
orm_model_pulihkan(db, U, "id", "7");        // pulihkan
biar aktif: teks = orm_model_semua_aktif(db, U);   // hanya yang belum dihapus

orm_cari_atau_buat(db, U, "nama", "Sri", peta{ "nama": "Sri", "umur": "33" });
orm_perbarui_atau_buat(db, U, "nama", "Sri", peta{ "umur": "34" });
```

## Catatan

- Satu driver aktif per koneksi (disimpan modul). Untuk dua DB beda driver bersamaan, gunakan modul `mysql`/`postgres` langsung.
- Lengkap: query builder berantai, transaksi, upsert/bulk, paginasi, agregasi, indeks/FK, relasi (termasuk many-to-many), soft-delete, dan validasi.

## Lisensi

MIT.
