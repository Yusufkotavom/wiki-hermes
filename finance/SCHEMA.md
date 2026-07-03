# Finance Wiki Schema

Domain: personal money tracking + proof + AR/AP.
Semua file finance melakukan append/logic di bawah folder ini. Jangan edit file di luar folder ini untuk data finance.

## Struktur

- `transactions.json` — transaksi pemasukan/pengeluaran
- `categories.json` — kategori pengeluaran/pemasukan
- `wallets.json` — dompet/vault/cash/bank
- `debts.json` — piutang/hutang
- `receipts/YYYY-MM/` — bukti transaksi
- `reports/YYYY-MM.md` — ringkasan bulanan

## Aturan

- Nominal disimpan dalam satuan dasar, int, tanpa koma/format.
- Mata uang default: `IDR`. Kalau beda, catat di field `currency`.
- Semua timestamp dalam `YYYY-MM-DD` atau ISO-8601 penuh.
- Tambah transaksi dengan generated `id` unik: `T-<date>-<seq>`.
- Setiap transaksi memiliki `type: income|expense|transfer|debt|repayment`.
- Setiap transaksi opsional `proof` berupa path relatif ke `receipts/...`
- Update file JSON dalam bentuk valid JSON, komentar tidak diizinkan.
- Update `reports/YYYY-MM.md` secara manual/otomatis sebagai ringkasan.
- Jangan commit file .env atau data sensitif; invoice/receipt boleh jika tidak mengandung kredensial sensitif.

## Kategori

- income: project, retainer, produk, investasi, hibah, lainnya
- expense: gaji, transport, makan, utilities, sewa, pemasaran, operasional, hardware, software, lainnya
- transfer: internal antar wallet
- debt: person, institutional
- repayment: person, institutional
