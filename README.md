# Public Sale Live Buy Bot (Webhook → Discord/Telegram)

Contoh proyek **Node.js** untuk menerima webhook `purchase.confirmed` dari **website public sale**, memverifikasi **HMAC**, melakukan **dedup**, lalu mengirim notifikasi ke **Discord** dan/atau **Telegram**.

## Fitur
- ✅ Verifikasi **HMAC-SHA256** (`X-Webhook-Signature`) terhadap **RAW body**
- ✅ Dedup (hindari pengumuman ganda) berbasis `orderId` (persist ke file)
- ✅ Format pesan **Discord embed** dan **Telegram message**
- ✅ Endpoint **/health** untuk monitoring
- ✅ Konversi USD → IDR (opsional)

## Prasyarat
- Node.js 18+
- Buat file `.env` berdasarkan `.env.example`

## Instalasi
```bash
cd buy-bot-public-sale
npm install
# salin .env contoh lalu edit
cp .env.example .env
# jalankan
npm start
```

## Endpoint
- **POST** `/webhook/public-sale`  
  Header: `Content-Type: application/json`  
  Header: `X-Webhook-Signature: <hmac_hex>`  
  Body (contoh):
  ```json
  {
    "event": "purchase.confirmed",
    "orderId": "ORD-2025-09-10-000123",
    "txHash": "0xabc123...",
    "buyer": "0xBEEF...CAFE",
    "tokenSymbol": "MNDL",
    "tokenDecimals": 18,
    "amountToken": "12345000000000000000000",
    "pricePerTokenUSD": "0.19",
    "paidCurrency": "USDC",
    "paidAmount": "2340.00",
    "timestamp": "2025-09-10T01:23:45Z",
    "explorerTxUrl": "https://explorer.example/tx/0xabc123..."
  }
  ```

- **GET** `/health` → `{"ok":true}`

## Cara buat signature di backend (contoh Node.js)
```js
const crypto = require('crypto');
const body = JSON.stringify(payload); // harus body mentah yang sama dengan yang dikirim
const signature = crypto.createHmac('sha256', process.env.WEBHOOK_SECRET)
  .update(body, 'utf8')
  .digest('hex');

await fetch(process.env.BUYBOT_WEBHOOK_URL, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-Webhook-Signature': signature,
    'X-Timestamp': new Date().toISOString()
  },
  body
});
```

## Uji lokal (contoh curl)
```bash
# Ganti SECRET sesuai .env, dan BODY sesuai payload JSON
BODY='{"event":"purchase.confirmed","orderId":"ORD-1","buyer":"0x1234","tokenSymbol":"MNDL","tokenDecimals":18,"amountToken":"1000000000000000000","pricePerTokenUSD":"0.2","timestamp":"2025-09-10T01:02:03Z","explorerTxUrl":"https://explorer/tx/0xabc"}'
SIG=$(python3 - <<'PY'
import hmac, hashlib, os, sys
secret = os.environ.get('SECRET','changeme').encode()
body = os.environ.get('BODY','').encode()
print(hmac.new(secret, body, hashlib.sha256).hexdigest())
PY
)
curl -i http://localhost:3000/webhook/public-sale   -H "Content-Type: application/json"   -H "X-Webhook-Signature: $SIG"   --data "$BODY"
```

## Catatan
- Proyek ini fokus untuk jalur **website → webhook** (bukan watcher on-chain).
- Untuk angka besar/precise, pertimbangkan pakai **decimal/bignumber** di produksi.
- Atur **whale threshold**, format brand, dsb di `src/index.js`.

---

© 2025 MandalaChain sample (MIT License)
# live_buy_bot
# live_buy_bot
