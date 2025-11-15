const {
    default: makeWASocket,
    useMultiFileAuthState
} = require("@whiskeysockets/baileys")
const fs = require("fs")

async function startWakataBot() {
    const { state, saveCreds } = await useMultiFileAuthState("./auth")

    const sock = makeWASocket({
        auth: state,
        printQRInTerminal: true
    })

    sock.ev.on("creds.update", saveCreds)

    sock.ev.on("messages.upsert", async ({ messages }) => {
        const m = messages[0]
        if (!m.message || !m.key.remoteJid) return
        
        const from = m.key.remoteJid
        const text = m.message.conversation || m.message.extendedTextMessage?.text

        if (!text) return

        // 📌 MENÚ
        if (text === "!menu") {
            let menu = `
┏━━━━━━━━━━ WAKATA ━━━━━━━━━━┓
┃  🤖 BOT MULTIFUNCIÓN
┃  
┃  ✦ Menú principal
┃  ✧ !ytmp3 — Música
┃  ✧ !ytmp4 — Video
┃  ✧ !todos — Mencionar a todos
┃  ✧ !info — Información
┃  
┃  ✦ Administración
┃  ✧ !promote @
┃  ✧ !demote @
┃  ✧ !kick @
┃  ✧ !add número
┃  
┃  🔗 Canal oficial:
┃  https://whatsapp.com/channel/0029Vb6bwLdFHWq2YyATLa2C
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
            `
            await sock.sendMessage(from, { text: menu })
        }

        // 📌 MENCIONAR A TODOS
        if (text === "!todos") {
            try {
                let grupo = await sock.groupMetadata(from)
                let participantes = grupo.participants.map(u => u.id)

                let msg = `🔥 *MENCIÓN MASIVA DE WAKATA* 🔥\n\n`
                msg += participantes.map(u => `@${u.split("@")[0]}`).join(" ")

                await sock.sendMessage(from, {
                    text: msg,
                    mentions: participantes
                })
            } catch (e) {
                await sock.sendMessage(from, { text: "❌ Este comando solo funciona en grupos." })
            }
        }

        // 📌 INFO
        if (text === "!info") {
            await sock.sendMessage(from, {
                text: `🤖 *WAKATA BOT*\n\nBot multifunción con administración, descargas y utilidades.\n\n🔗 Canal oficial:\nhttps://whatsapp.com/channel/0029Vb6bwLdFHWq2YyATLa2C`
            })
        }

    })
}

startWakataBot()
