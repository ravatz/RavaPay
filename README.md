<div align="center">

<img src="./.github/assets/icon.svg" alt="RavaPay Logo" width="120" height="120" />
<br/>

# RavaPay

**Platform otomasi pembayaran QRIS multi-provider dengan API yang powerful untuk merchant Indonesia**

[![Website](https://img.shields.io/badge/Website-ravapay.site-blue)](https://ravapay.site)
[![Telegram](https://img.shields.io/badge/Telegram-@RavaPayBot-blue)](https://t.me/RavaPayBot)
<br/>
[![API Status](https://img.shields.io/uptimerobot/status/m803062190-f62349fffaae470837b92a9c?label=API)](https://stats.uptimerobot.com/0y8nSx5FN2)
[![API Uptime](https://img.shields.io/uptimerobot/ratio/30/m803062190-f62349fffaae470837b92a9c?label=API%20uptime%2030d)](https://stats.uptimerobot.com/0y8nSx5FN2)
[![Dashboard Status](https://img.shields.io/uptimerobot/status/m803062182-82f4aa15991eb4a440d58c6d?label=Dashboard)](https://stats.uptimerobot.com/0y8nSx5FN2)
[![Dashboard Uptime](https://img.shields.io/uptimerobot/ratio/30/m803062182-82f4aa15991eb4a440d58c6d?label=Dashboard%20uptime%2030d)](https://stats.uptimerobot.com/0y8nSx5FN2)

</div>

---

RavaPay adalah platform yang memudahkan merchant menerima pembayaran QRIS otomatis melalui satu REST API. Satu integrasi bisa dipakai untuk beberapa provider seperti GoPay dan BRI. Dengan RavaPay, Anda dapat:

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

Data terenkripsi dengan standar industri. Kredensial provider Anda disimpan dengan enkripsi penuh — tidak pernah tersimpan dalam bentuk plaintext.

---

### Diagram Alur Sistem

```mermaid
sequenceDiagram
    participant Merchant as 🏪 Merchant/Your System
    participant RavaPay as ⚡ RavaPay API
    participant Provider as 🏦 Provider QRIS
    participant Customer as 👤 Customer

    Note over Merchant,Customer: 1️⃣ Generate QRIS
    Merchant->>RavaPay: POST /create<br/>{provider: "gopay", amount: 50000}
    RavaPay->>Provider: Request QRIS via provider account
    Provider-->>RavaPay: Return QRIS data & transaction ID
    RavaPay-->>Merchant: {provider, transaction_id, qr_string, qr_url}

    Note over Merchant,Customer: 2️⃣ Customer Melakukan Pembayaran
    Merchant->>Customer: Tampilkan QRIS ke customer
    Customer->>Provider: Scan QRIS & bayar via aplikasi pembayaran
    Provider-->>Customer: ✅ Pembayaran Berhasil

    Note over Merchant,Customer: 3️⃣ Notifikasi Real-time
    Provider->>RavaPay: Payment Notification / Mutation Polling
    RavaPay->>RavaPay: Verify & Process Payment
    RavaPay->>Merchant: 🔔 POST Webhook Callback<br/>{event: "payment.success", provider, amount: 50000}
    Merchant-->>RavaPay: 200 OK (Webhook Received)

    Note over Merchant,Customer: 4️⃣ Update Dashboard
    RavaPay->>RavaPay: Update Transaction Status
    Merchant->>RavaPay: GET /transactions/:id
    RavaPay-->>Merchant: {provider, status: "success", payment_reference: "..."}
```

---

### Alur Transaksi Step-by-Step

#### 1️⃣ Generate QRIS

Sistem Anda me-request QRIS dengan nominal tertentu melalui API RavaPay. Pilih provider lewat field `provider` (`gopay` atau `bri`), lalu RavaPay akan meneruskan request ke provider terkait dan mengembalikan QRIS beserta Transaction ID.

**Request:**

```bash
curl -X POST https://api.ravapay.site/create \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"provider": "gopay", "amount": 50000, "description": "Order #1234"}'
```

**Response:** `201 Created`

```json
{
  "success": true,
  "message": "Payment QRIS created successfully",
  "data": {
    "provider": "gopay",
    "transaction_id": "TRX-FD8492591A2B4C6D",
    "amount": 50000,
    "status": "pending",
    "qr_string": "00020101021226...",
    "qr_url": "https://api.ravapay.site/qr/TRX-FD8492591A2B4C6D",
    "created_at": "2026-05-01T10:15:00.000Z",
    "expired_at": "2026-05-01T10:30:00.000Z"
  }
}
```

#### 2️⃣ Customer Bayar

Customer melakukan scan QRIS (via URL dari `qr_url` atau `qr_string`) menggunakan aplikasi pembayaran yang mendukung QRIS, lalu menyelesaikan pembayaran.

#### 3️⃣ Webhook Callback (Real-time)

Segera setelah pembayaran terkonfirmasi oleh provider, RavaPay akan mengirim Webhook langsung ke server Anda.

**Diagram Webhook Flow**
```mermaid
sequenceDiagram
    participant Customer as 👤 Customer
    participant Provider as 🏦 Provider QRIS
    participant RavaPay as ⚡ RavaPay API
    participant YourSystem as 🏪 Your System
    participant Dashboard as 📊 Dashboard

    Note over Customer,Dashboard: Customer melakukan pembayaran
    Customer->>Provider: 1. Scan QRIS & Input PIN
    Provider->>Provider: 2. Process Payment
    Provider-->>Customer: 3. ✅ Payment Success

    Note over Customer,Dashboard: RavaPay menerima notifikasi
    Provider->>RavaPay: 4. Payment Notification / Mutation Polling
    RavaPay->>RavaPay: 5. Verify Payment Data

    Note over Customer,Dashboard: Webhook dikirim ke sistem Anda
    RavaPay->>YourSystem: 6. 🔔 POST Webhook<br/>{event, provider, transaction_id}
    YourSystem->>YourSystem: 7. Process Order/Update Status
    YourSystem-->>RavaPay: 8. 200 OK (Acknowledge)

    Note over Customer,Dashboard: Dashboard terupdate otomatis
    RavaPay->>Dashboard: 9. Update Transaction Status
```

**Webhook Payload:**

```json
{
  "event": "payment.success",
  "data": {
    "transaction_id": "TRX-FD8492591A2B4C6D",
    "provider": "gopay",
    "status": "success",
    "amount": 50000,
    "description": "Order #1234",
    "customer_name": null,
    "customer_phone": null,
    "customer_email": null,
    "qr_string": "00020101021226...",
    "qr_url": "https://api.ravapay.site/qr/TRX-FD8492591A2B4C6D",
    "created_at": "2026-05-01T10:15:00.000Z",
    "expired_at": "2026-05-01T10:30:00.000Z",
    "updated_at": "2026-05-01T10:17:43.000Z"
  },
  "timestamp": "2026-05-01T10:17:43.000Z"
}
```

> 📌 **Event types:** `payment.success` · `payment.expired` · `payment.cancel`
> Webhook dipicu setiap kali status transaksi berubah ke salah satu state di atas.

#### 🔐 Verifikasi Webhook Signature

Setiap webhook dilengkapi **HMAC-SHA256 signature** pada header `X-RavaPay-Signature` untuk keamanan. Gunakan **Webhook Secret** Anda (dapatkan di Dashboard) untuk memverifikasi keaslian pengirim:

```javascript
// Node.js (Express) Example — gunakan raw body middleware
const crypto = require('crypto');
const express = require('express');
const app = express();

// PENTING: signature divalidasi atas RAW BODY, bukan JSON yang sudah di-parse.
// Pakai express.raw() agar req.body berupa Buffer yang belum tersentuh.
app.post(
  '/webhook/ravapay',
  express.raw({ type: 'application/json' }),
  (req, res) => {
    const signature = req.headers['x-ravapay-signature'];
    const rawBody = req.body; // Buffer
    const webhookSecret = process.env.RAVAPAY_WEBHOOK_SECRET;

    const expectedSignature = crypto
      .createHmac('sha256', webhookSecret)
      .update(rawBody)
      .digest('hex');

    // Bandingkan dengan timing-safe comparison
    const sigBuf = Buffer.from(signature || '', 'hex');
    const expBuf = Buffer.from(expectedSignature, 'hex');
    if (sigBuf.length !== expBuf.length || !crypto.timingSafeEqual(sigBuf, expBuf)) {
      return res.status(401).json({ error: 'Invalid signature' });
    }

    // Parse body setelah signature valid
    const payload = JSON.parse(rawBody.toString('utf8'));

    switch (payload.event) {
      case 'payment.success':
        console.log(`Pembayaran berhasil: ${payload.data.transaction_id}`);
        // TODO: Update order di database Anda
        break;
      case 'payment.expired':
        console.log(`Transaksi expired: ${payload.data.transaction_id}`);
        // TODO: Release stock, tandai order sebagai kadaluarsa
        break;
      case 'payment.cancel':
        console.log(`Transaksi dibatalkan: ${payload.data.transaction_id}`);
        break;
    }

    res.status(200).json({ success: true });
  }
);
```

```php
<?php
// PHP Example — baca raw body dari stdin
$payload = file_get_contents('php://input');
$signature = $_SERVER['HTTP_X_RAVAPAY_SIGNATURE'] ?? '';
$webhookSecret = getenv('RAVAPAY_WEBHOOK_SECRET');

// Generate signature pembanding (raw hex, tanpa prefix)
$expectedSignature = hash_hmac('sha256', $payload, $webhookSecret);

// Verifikasi signature dengan timing-safe comparison
if (!hash_equals($expectedSignature, $signature)) {
    http_response_code(401);
    exit('Invalid signature');
}

$data = json_decode($payload, true);

switch ($data['event']) {
    case 'payment.success':
        $transactionId = $data['data']['transaction_id'];
        // TODO: Update status order di database Anda
        break;
    case 'payment.expired':
        // TODO: Release stock, tandai order sebagai kadaluarsa
        break;
    case 'payment.cancel':
        // TODO: Handle transaksi yang dibatalkan
        break;
}

http_response_code(200);
echo json_encode(['success' => true]);
?>
```

#### 4️⃣ Cek Status (Opsional)

Anda juga bisa memverifikasi status secara manual via API:

**Request:**

```bash
curl https://api.ravapay.site/transactions/TRX-FD8492591A2B4C6D \
  -H "x-api-key: YOUR_API_KEY"
```

**Response:**

```json
{
  "success": true,
  "message": "Get transaction successful",
  "data": {
    "provider": "gopay",
    "transaction_id": "TRX-FD8492591A2B4C6D",
    "amount": 50000,
    "status": "success",
    "payment_reference": "f8a7b3c2-d4e5-...",
    "customer_name": null,
    "created_at": "2026-05-01T10:15:00.000Z",
    "expired_at": "2026-05-01T10:30:00.000Z"
  }
}
```

> 📌 Status valid: `pending` · `success` · `expired` · `cancel`
> `payment_reference` bernilai `null` selama transaksi belum terbayar.

---

## 🚀 Cara Memulai

#### 1️⃣ Registrasi & Login

Daftar akun secara gratis di [ravapay.site/register](https://ravapay.site) lalu login ke Dashboard.
**Persyaratan:**

- 👤 Username
- 🔐 Password yang kuat

#### 2️⃣ Hubungkan Provider QRIS

- Di dalam dashboard, navigasikan ke menu koneksi provider.
- Hubungkan akun provider yang tersedia, seperti GoPay Merchant atau BRI Merchant.
- Kredensial disimpan terenkripsi dan tidak pernah tersimpan dalam bentuk plaintext.

#### 3️⃣ Langganan Paket (Subscription)

Langganan dikelola sepenuhnya oleh sistem Bot Telegram kami. Anda bisa melakukannya dengan dua cara:
- **Via Dashboard**: Buka halaman **Pricing** dan pilih paket yang sesuai. Jika Anda belum login, sistem akan mengarahkan ke halaman login/register dulu. Setelah login, klik paket akan membuka [@RavaPayBot](https://t.me/RavaPayBot) via _deep link_ dengan paket yang sudah terpilih otomatis.
- **Via Bot Langsung**: Buka [@RavaPayBot](https://t.me/RavaPayBot) dan ikuti instruksi pembelian paket di dalam bot.
- Selesaikan pembayaran invoice QRIS yang diberikan bot, dan langganan Anda akan otomatis aktif!

#### 4️⃣ Setup API Key & Webhook

- Di menu **Settings**, klik **Generate API Key**.
- Masukkan **Webhook URL** Anda.
- Ambil **Webhook Secret** untuk verifikasi signature (HMAC-SHA256).

#### 5️⃣ Mulai Terima Pembayaran

- Integrasikan API RavaPay ke sistem Anda mengikuti [Dokumentasi API interaktif](https://ravapay.site/docs) di dashboard.
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

Semua data sensitif termasuk kredensial provider disimpan menggunakan enkripsi tingkat tinggi. **Kami tidak pernah menyimpan informasi ini dalam bentuk plaintext**.

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

### 📊 Apakah ada limit transaksi provider?

Tidak ada batasan dari sistem RavaPay. Limit mutasi, settlement, dan pencairan sepenuhnya bergantung pada kebijakan tier akun provider Anda.

### 📱 Apakah bisa pakai akun personal?

Sistem RavaPay didesain untuk akun merchant/provider bisnis, seperti **GoPay Merchant** atau **BRI Merchant**. Pastikan Anda menggunakan akun merchant untuk kinerja maksimal.

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
