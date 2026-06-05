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
      skema.tenun       orm_buat_tabel / orm_hapus_tabel / orm_tambah_kolom
      query.tenun       orm_hitung / orm_cari_satu / orm_urut
    relasi/
      relasi.tenun      orm_punya_banyak / orm_milik / orm_gabung / orm_gabung_cari
    model/
      model.tenun       definisi model (peta): orm_model / orm_kolom / orm_migrasi / orm_simpan / orm_baris_peta
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

Fungsi model: `orm_model`, `orm_kolom`, `orm_model_kolom`, `orm_migrasi`, `orm_model_hapus_tabel`, `orm_simpan`, `orm_model_semua`, `orm_model_cari`, `orm_model_hapus`, `orm_baris_peta`. Semua nilai tetap berparameter (aman injeksi); diuji live di MySQL/MariaDB & PostgreSQL.

## Catatan

- Satu driver aktif per koneksi (disimpan modul). Untuk dua DB beda driver bersamaan, gunakan modul `mysql`/`postgres` langsung.
- Definisi model, relasi, migrasi, dan CRUD aman sudah tersedia. JOIN builder lanjutan menyusul.

## Lisensi

MIT.
