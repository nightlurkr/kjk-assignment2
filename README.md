# Assignment 2 — Zone-Based Firewall

Network Security (ET234302) · Institut Teknologi Sepuluh Nopember
Kelompok NS01 · lab `vmw-ns-01` · 16 November 2026

## Isi repo

| File | Keterangan |
|---|---|
| `arsitektur.html` | Laporan satu halaman. Buka di browser, Ctrl+P, Save as PDF. Ini yang dikumpulkan. |
| `arsitektur.pdf` | Hasil cetak dari file di atas. |
| `rules/` | Salinan `/etc/lab/r1/`, `/etc/lab/r2/`, `/etc/lab/db/`. |
| `bukti/` | Output `nft list ruleset`, log drop, dan hasil probe. |

## Topologi

Lima zona, dua router, dan firewall host di database.

| Zona | Interface | Subnet | Host |
|---|---|---|---|
| OUTSIDE | `out0` | 203.0.113.0/24 | internet .10, attacker .66 |
| INSIDE | `in0` | 192.168.10.0/24 | staff .50 |
| MGMT | `mgmt0` | 192.168.99.0/24 | bastion .5, admin .10 |
| DMZ | `dmz0` | 172.16.10.0/24 | web .10 |
| DATA | `data0` | 172.16.20.0/24 | db .10 |

| Router | Interface dan IP |
|---|---|
| r1 | out0 203.0.113.1 · in0 192.168.10.1 · mgmt0 192.168.99.1 · dmz0 172.16.10.1 |
| r2 | dmz0 172.16.10.2 · mgmt0 192.168.99.2 · data0 172.16.20.1 |

Host `db` menjalankan rulesetnya sendiri (`table inet db`), jadi database terlindungi dua lapis: oleh r2 dan oleh dirinya sendiri.

## Prinsip yang dipakai

- Semua chain memakai `policy drop`. Yang tidak diizinkan secara eksplisit akan dibuang.
- `ct state established,related accept` di baris atas tiap chain, supaya balasan kembali tanpa membuka rule lebar.
- `ct state invalid counter drop` membuang paket yang tidak masuk akal.
- Chain `antispoof` di r1 menolak alamat bogon dari luar dan memeriksa source tiap interface.
- Database tidak pernah memulai koneksi keluar.
- DMZ tidak boleh memulai koneksi ke INSIDE maupun MGMT.
- Akses administratif hanya lewat bastion 192.168.99.5.
- Tiap chain ditutup `log prefix ... counter drop` supaya drop bisa dibuktikan.

## Cara memuat rule

```bash
sudo lab r1 nft -f /etc/lab/r1/r1.nft
sudo lab r2 nft -f /etc/lab/r2/r2.nft
sudo lab db nft -f /etc/lab/db/db.nft
```

## Cara memeriksa

```bash
sudo lab r1 nft list ruleset
sudo lab r2 nft list ruleset
sudo lab db nft list ruleset
```

Counter drop muncul di output yang sama. Jalankan probe dulu, baru lihat counternya naik.

## Catatan integritas

Rule ditulis dan diketik sendiri oleh anggota kelompok. AI dipakai sebatas tanya jawab di laptop, sesuai aturan tugas: tidak ada AI agent yang menjalankan perintah di server, port 22 tidak dibuka, lab guard dan Teleport tidak diubah.
