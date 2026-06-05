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
  tenun.json          (butuh: mysql, postgres)
  src/
    orm.tenun         entry — impor mysql, postgres, dan berkas di bawah
    koneksi.tenun     orm_sambung / orm_tutup
    kueri.tenun       orm_jalan / orm_kueri / orm_baris / orm_ambil
    crud.tenun        orm_semua / orm_cari / orm_sisip / orm_ubah / orm_hapus
```

## Catatan

- Satu driver aktif per koneksi (disimpan modul). Untuk dua DB beda driver bersamaan, gunakan modul `mysql`/`postgres` langsung.
- Fitur lanjutan (definisi model, relasi, migrasi, JOIN builder) menyusul.

## Lisensi

MIT.
