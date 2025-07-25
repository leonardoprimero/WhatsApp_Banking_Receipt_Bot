# 📲 WhatsApp Banking Receipt Bot: Effortless Automation with n8n + Google Sheets

![n8n Transfer Workflow](https://firebasestorage.googleapis.com/v0/b/leonardo-blog.firebasestorage.app/o/n8ntransferencias.webp?alt=media&token=7f529d61-f3ac-4a48-858d-b0572a787142)

Have you ever wished your banking receipts would just *magically* organize themselves?  
Welcome to the future: this WhatsApp bot receives payment/transfer receipts (PDF, JPG, PNG), extracts all the key data with OCR and AI, and logs them straight into your Google Sheets.  
All in real time, 24/7, from your cloud server, no manual input, no more chaos.  
Perfect for freelancers, small businesses, or anyone sick of chasing paperwork.

## ✨ How It Works

1. **You send a transfer receipt via WhatsApp** (as PDF or image).
2. The bot listens for new messages with media, checks the file type and downloads it.
3. The file is sent via webhook to an n8n workflow, where:
   - OCR (Google Vision, OCR.Space, etc) extracts the text.
   - OpenAI parses all relevant data: date, amount, origin account, CBU/CUIL, bank, sender, etc.
   - All this data is appended to your Google Sheet, including a clickable WhatsApp link.
4. You instantly have your transactions logged and accessible — no matter where you are.

And yes: you could hook this up to Telegram, Drive, Dropbox, wherever you like.

## 🚀 Quick Glimpse – The WhatsApp Bot (index.js)

```js
const { Client, LocalAuth } = require('whatsapp-web.js');
const qrcode = require('qrcode-terminal');
const axios = require('axios');

const colaComprobantes = [];
let procesando = false;

async function procesarCola() {
  if (procesando) return;
  procesando = true;
  while (colaComprobantes.length > 0) {
    const { media, msg, texto } = colaComprobantes.shift();
    try {
      await axios.post('https://YOUR_WEBHOOK_HERE', {
        fileName: media.filename,
        data: media.data,
        mimetype: media.mimetype,
        from: msg.from,
        nombreOrigen: msg._data.notifyName || '',
        textoCompleto: texto,
        fecha: new Date().toISOString()
      });
      console.log('✅ Processed:', media.filename);
    } catch (e) {
      console.error('❌ Error processing media:', e);
    }
    await new Promise(r => setTimeout(r, 2000));
  }
  procesando = false;
}

const client = new Client({
  authStrategy: new LocalAuth({ clientId: 'bot-transferencias' }),
  puppeteer: { args: ['--no-sandbox'] }
});

client.on('qr', qr => {
  qrcode.generate(qr, { small: true });
  console.log('Scan this QR code to link WhatsApp!');
});

client.on('ready', () => {
  console.log('✅ WhatsApp bot connected.');
});

client.on('message', async msg => {
  if (msg.from.endsWith('@g.us')) return;
  if (msg.hasMedia) {
    try {
      const media = await msg.downloadMedia();
      if (!media) return;
      const validTypes = ['application/pdf', 'image/jpeg', 'image/png', 'image/jpg'];
      if (validTypes.includes(media.mimetype)) {
        colaComprobantes.push({ media, msg, texto: msg.body || '' });
        procesarCola();
      }
    } catch (e) {
      console.error('❌ Error:', e);
    }
  }
});

client.initialize();
```


## 🛠️ What You Need

- WhatsApp account and [`whatsapp-web.js`](https://github.com/pedroslopez/whatsapp-web.js)
- [`n8n.io`](https://n8n.io/) running on a cloud server (or local)
- Google Sheet (with Google API credentials)
- OCR API key (OCR.Space, Google Vision, etc)
- OpenAI API key (for smarter parsing)

---

## 🧠 Why Bother?

Because time is money. And wasting it copying numbers by hand is so 2010.  
This system saves you hours, avoids mistakes, and gives you peace of mind — receipts organized, always accessible, zero paperwork.

> Life is too short to spend it typing CBU numbers.

---

Feel free to fork, adapt, or drop me a line if you want to take your automation further.

**leonardoprimero a.k.a. Leonardo I**

