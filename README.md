<div align="center">

<img src="./.github/assets/icon.svg" alt="RavaPay Logo" width="120" height="120" />
<br/>

# RavaPay

**Platform otomasi pembayaran QRIS dengan API yang powerful untuk merchant Indonesia**

[![Website](https://img.shields.io/badge/Website-ravapay.biz.id-blue)](https://ravapay.biz.id)
[![Telegram](https://img.shields.io/badge/Telegram-@RavaPayBot-blue)](https://t.me/RavaPayBot)
<br/>
[![API Status](https://img.shields.io/uptimerobot/status/m803062190-f62349fffaae470837b92a9c?label=API)](https://stats.uptimerobot.com/0y8nSx5FN2)
[![API Uptime](https://img.shields.io/uptimerobot/ratio/30/m803062190-f62349fffaae470837b92a9c?label=API%20uptime%2030d)](https://stats.uptimerobot.com/0y8nSx5FN2)
[![Dashboard Status](https://img.shields.io/uptimerobot/status/m803062182-82f4aa15991eb4a440d58c6d?label=Dashboard)](https://stats.uptimerobot.com/0y8nSx5FN2)
[![Dashboard Uptime](https://img.shields.io/uptimerobot/ratio/30/m803062182-82f4aa15991eb4a440d58c6d?label=Dashboard%20uptime%2030d)](https://stats.uptimerobot.com/0y8nSx5FN2)

</div>

---

RavaPay adalah platform yang memudahkan merchant untuk menerima pembayaran GoPay secara otomatis melalui QRIS. Dengan RavaPay, Anda dapat:

- Generate QRIS dinamis dengan nominal custom
- Cek status transaksi secara real-time
- Terima notifikasi otomatis via webhook
- Monitor semua transaksi di dashboard merchant
- Integrasi mudah dengan sistem Anda via REST API

---

### Generate QRIS Otomatis

Buat QRIS dinamis dengan nominal custom untuk setiap transaksi. Setiap QRIS memiliki Transaction ID unik untuk tracking yang akurat.

### Cek Status Real-time

Pantau status pembayaran secara real-time menggunakan Transaction ID. Tidak perlu khawatir dengan transaksi yang memiliki nominal sama.

### Webhook Callback

Terima notifikasi otomatis ke sistem Anda saat transaksi berhasil. Payload dikirim dengan tanda tangan HMAC-SHA256 untuk memastikan keaslian event.

### Dashboard Monitoring

Dashboard intuitif untuk monitoring semua transaksi, statistik pendapatan, dan performa bisnis Anda secara real-time.

### Keamanan & Keandalan

Data terenkripsi dengan standar industri. Kredensial GoPay Anda disimpan dengan enkripsi penuh — tidak pernah tersimpan dalam bentuk plaintext.

---

### Diagram Alur Sistem

```mermaid
sequenceDiagram
    participant Merchant as 🏪 Merchant/Your System
    participant RavaPay as ⚡ RavaPay API
    participant GoPay as 💳 GoPay Server
    participant Customer as 👤 Customer

    Note over Merchant,Customer: 1️⃣ Generate QRIS
    Merchant->>RavaPay: POST /gopay/create<br/>{amount: 50000}
    RavaPay->>GoPay: Request QRIS via merchant account
    GoPay-->>RavaPay: Return QRIS data & transaction ID
    RavaPay-->>Merchant: {transaction_id, qr_string, qr_url}

    Note over Merchant,Customer: 2️⃣ Customer Melakukan Pembayaran
    Merchant->>Customer: Tampilkan QRIS ke customer
    Customer->>GoPay: Scan QRIS & Bayar via GoPay App
    GoPay-->>Customer: ✅ Pembayaran Berhasil

    Note over Merchant,Customer: 3️⃣ Notifikasi Real-time
    GoPay->>RavaPay: Payment Success Notification
    RavaPay->>RavaPay: Verify & Process Payment
    RavaPay->>Merchant: 🔔 POST Webhook Callback<br/>{status: "SUCCESS", amount: 50000}
    Merchant-->>RavaPay: 200 OK (Webhook Received)

    Note over Merchant,Customer: 4️⃣ Update Dashboard
    RavaPay->>RavaPay: Update Transaction Status
    Merchant->>RavaPay: GET /gopay/transactions/:id
    RavaPay-->>Merchant: {status: "SUCCESS", paid_at: "..."}
```

---

### Alur Transaksi Step-by-Step

#### 1️⃣ Generate QRIS

Sistem Anda me-request QRIS dengan nominal tertentu melalui API. RavaPay akan meneruskan request ke server GoPay dan mengembalikan QRIS beserta Transaction ID.

**Request:**

```bash
curl -X POST https://api.ravapay.biz.id/gopay/create \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"amount": 50000, "remarks": "Order #1234"}'
```

**Response:**

```json
{
  "success": true,
  "data": {
    "transaction_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "amount": 50000,
    "status": "PENDING",
    "qr_string": "00020101021126...",
    "qr_url": "https://api.midtrans.com/qr/...",
    "created_at": "2026-05-01T10:15:00Z",
    "expires_at": "2026-05-01T10:30:00Z"
  }
}
```

#### 2️⃣ Customer Bayar

Customer melakukan scan QRIS (via URL dari `qr_url` atau `qr_string`) menggunakan aplikasi GoPay dan menyelesaikan pembayaran.

#### 3️⃣ Webhook Callback (Real-time)

Segera setelah GoPay menyatakan pembayaran berhasil, RavaPay akan mengirim Webhook langsung ke server Anda.

**Diagram Webhook Flow**
```mermaid
sequenceDiagram
    participant Customer as 👤 Customer
    participant GoPay as 💳 GoPay App
    participant RavaPay as ⚡ RavaPay API
    participant YourSystem as 🏪 Your System
    participant Dashboard as 📊 Dashboard

    Note over Customer,Dashboard: Customer melakukan pembayaran
    Customer->>GoPay: 1. Scan QRIS & Input PIN
    GoPay->>GoPay: 2. Process Payment
    GoPay-->>Customer: 3. ✅ Payment Success

    Note over Customer,Dashboard: RavaPay menerima notifikasi
    GoPay->>RavaPay: 4. Payment Notification
    RavaPay->>RavaPay: 5. Verify Payment Data

    Note over Customer,Dashboard: Webhook dikirim ke sistem Anda
    RavaPay->>YourSystem: 6. 🔔 POST Webhook<br/>{event, transaction_id}
    YourSystem->>YourSystem: 7. Process Order/Update Status
    YourSystem-->>RavaPay: 8. 200 OK (Acknowledge)

    Note over Customer,Dashboard: Dashboard terupdate otomatis
    RavaPay->>Dashboard: 9. Update Transaction Status
```

**Webhook Payload:**

```json
{
  "event": "transaction.completed",
  "timestamp": "2026-05-01T10:17:43Z",
  "data": {
    "transaction_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "amount": 50000,
    "status": "SUCCESS",
    "remarks": "Order #1234"
  }
}
```

#### 🔐 Verifikasi Webhook Signature

Setiap webhook dilengkapi **HMAC-SHA256 signature** pada header `X-RavaPay-Signature` untuk keamanan. Gunakan **Webhook Secret** Anda (dapatkan di Dashboard) untuk memverifikasi keaslian pengirim:

```javascript
// Node.js (Express) Example
const crypto = require('crypto');

app.post('/webhook/ravapay', (req, res) => {
  // Ambil signature dari header
  const signature = req.headers['x-ravapay-signature'];
  
  // Pastikan Anda menggunakan raw body dari request
  const rawBody = JSON.stringify(req.body); 
  const webhookSecret = 'YOUR_WEBHOOK_SECRET';

  const expectedSignature = crypto
    .createHmac('sha256', webhookSecret)
    .update(rawBody)
    .digest('hex');

  // Verifikasi signature
  if (`sha256=${expectedSignature}` !== signature) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  // Proses event
  if (req.body.event === 'transaction.completed') {
    const { transaction_id, status } = req.body.data;
    console.log(`Payment success: ${transaction_id}`);
    // TODO: Update status order di database Anda
  }

  res.status(200).json({ success: true });
});
```

```php
// PHP Example
<?php
$payload = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_RAVAPAY_SIGNATURE'];
$webhookSecret = 'YOUR_WEBHOOK_SECRET';

// Generate signature pembanding
$expectedSignature = 'sha256=' . hash_hmac('sha256', $payload, $webhookSecret);

// Verifikasi signature
if (!hash_equals($expectedSignature, $signature)) {
    http_response_code(401);
    exit('Invalid signature');
}

$data = json_decode($payload, true);

// Proses event
if ($data['event'] === 'transaction.completed') {
    $transactionId = $data['data']['transaction_id'];
    // TODO: Update status order di database Anda
}

// Berikan respon 200 OK
http_response_code(200);
echo json_encode(['success' => true]);
?>
```

#### 4️⃣ Cek Status (Opsional)

Anda juga bisa memverifikasi status secara manual via API:

**Request:**

```bash
curl https://api.ravapay.biz.id/gopay/transactions/a1b2c3d4-e5f6-7890-abcd-ef1234567890 \
  -H "x-api-key: YOUR_API_KEY"
```

**Response:**

```json
{
  "success": true,
  "data": {
    "transaction_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "amount": 50000,
    "status": "SUCCESS",
    "paid_at": "2026-05-01T10:17:43Z"
  }
}
```

---

## 🚀 Cara Memulai

#### 1️⃣ Registrasi & Login

Daftar akun secara gratis di [ravapay.biz.id/register](https://ravapay.biz.id) lalu login ke Dashboard.
**Persyaratan:**

- 👤 Username
- 🔐 Password yang kuat

#### 2️⃣ Hubungkan GoPay Merchant

- Di dalam dashboard, navigasikan ke menu sinkronisasi GoPay.
- Masukkan kredensial GoPay Merchant Anda secara aman (Data dienkripsi).

#### 3️⃣ Langganan Paket (Subscription)

Langganan dikelola sepenuhnya oleh sistem Bot Telegram kami. Anda bisa melakukannya dengan dua cara:
- **Via Dashboard**: Buka halaman **Pricing** dan pilih paket yang sesuai. Anda akan diarahkan ke Telegram Bot ([@RavaPayBot](https://t.me/RavaPayBot)) via _deep link_.
- **Via Bot Langsung**: Buka [@RavaPayBot](https://t.me/RavaPayBot) dan ikuti instruksi pembelian paket di dalam bot.
- Selesaikan pembayaran invoice QRIS yang diberikan bot, dan langganan Anda akan otomatis aktif!

#### 4️⃣ Setup API Key & Webhook

- Di menu **Settings**, klik **Generate API Key**.
- Masukkan **Webhook URL** Anda.
- Ambil **Webhook Secret** untuk verifikasi signature (HMAC-SHA256).

#### 5️⃣ Mulai Terima Pembayaran

- Integrasikan API RavaPay ke sistem Anda mengikuti [Dokumentasi API interaktif](https://ravapay.biz.id/docs) di dashboard.
- Pantau semua transaksi di halaman dashboard utama secara live.

---

## 🛒 Use Cases

### E-Commerce

Terima pembayaran otomatis untuk toko online Anda. Webhook RavaPay langsung meng-update status order, sehingga customer tidak perlu konfirmasi manual.

### Donation Platform

Buat QRIS untuk donasi dengan nominal spesifik. Semua mutasi akan terlacak di dashboard dengan detail yang akurat.

### Layanan Berlangganan (SaaS)

Otomatiskan penagihan layanan bulanan menggunakan sistem RavaPay yang terintegrasi dengan invoice.

---

## 📊 Fitur Dashboard RavaPay

- 💰 **Total Pendapatan** - Analitik pendapatan harian, mingguan, bulanan.
- 📈 **Volume Transaksi** - Metrik pergerakan pembayaran secara live.
- 📋 **Riwayat Transaksi** - Daftar transaksi lengkap dengan modal detail & pagination halus.
- ⚙️ **Manajemen Keamanan** - Regenerasi API key dan rotasi Webhook secret secara mandiri.
- 📖 **API Docs** - Swagger UI internal untuk ujicoba API langsung dari browser.

---

## 🛡️ Keamanan & Privasi

### 🔐 Enkripsi Kredensial

Semua data sensitif termasuk kredensial GoPay disimpan menggunakan enkripsi tingkat tinggi. **Kami tidak pernah menyimpan informasi ini dalam bentuk plaintext**.

### ✍️ Webhook Signature

Setiap notifikasi webhook ditandatangani dengan HMAC-SHA256. Gunakan `Webhook Secret` Anda untuk memverifikasi payload agar terhindar dari pemalsuan request.

### 🚦 Rate Limiting

Sistem API dilengkapi throttling per-IP dan per-API-key untuk perlindungan ekstra dari serangan DDoS atau abusive requests.

---

## 🙋‍♂️ FAQ (Tanya Jawab)

### 💵 Berapa biaya layanannya?

Kami menggunakan model subscription (SaaS) berbasis langganan bulanan tanpa potongan per transaksi. Info lengkap dapat dilihat via Telegram bot.

### 🔄 Apakah bisa refund paket subscription?

Paket langganan yang sudah diaktifkan tidak dapat direfund.

### 📊 Apakah ada limit transaksi GoPay?

Tidak ada batasan dari sistem RavaPay. Limit mutasi dan pencairan sepenuhnya bergantung pada kebijakan tier akun GoPay Merchant Anda.

### 📱 Apakah bisa pakai akun GoPay personal?

Sistem RavaPay didesain khusus untuk mendukung akun **GoPay Merchant**. Pastikan Anda menggunakan nomor merchant untuk kinerja maksimal.

---

## 🆚 Kenapa Memilih RavaPay?

| Fitur                 | ⚡ RavaPay                           | 🐌 Manual                                  |
| --------------------- | ------------------------------------ | ------------------------------------------ |
| Generate QRIS         | ✅ Otomatis via API (Instant)        | ❌ Buka aplikasi & ketik satu per satu     |
| Notifikasi Pembayaran | ✅ Real-time Webhook                 | ❌ Tunggu notif & verifikasi mutasi manual |
| Monitoring Dashboard  | ✅ Terintegrasi (Analitik & Riwayat) | ❌ Terbatas pada mutasi bawaan aplikasi    |
| Integrasi Sistem      | ✅ REST API & Dokumentasi Swagger    | ❌ Tidak ada API                           |
| Keamanan Akses        | ✅ JWT + HMAC-SHA256 Signatures      | -                                          |

---

<div align="center">

Dibuat dengan ❤️ oleh **Tim RavaPay**

_Cepat · Aman · Developer-Friendly_

[![Telegram](https://img.shields.io/badge/Hubungi%20Kami-Telegram-blue?logo=telegram)](https://t.me/ravadigital)

</div>
