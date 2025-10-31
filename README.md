import makeWASocket, { useMultiFileAuthState } from '@whiskeysockets/baileys'
import fs from 'fs'

const startBot = async () => {
  const { state, saveCreds } = await useMultiFileAuthState('./auth')
  const sock = makeWASocket({
    auth: state,
    printQRInTerminal: true
  })

  sock.ev.on('creds.update', saveCreds)

  sock.ev.on('messages.upsert', async (m) => {
    const msg = m.messages[0]
    if (!msg.message) return
    const from = msg.key.remoteJid
    const text = msg.message.conversation || msg.message.extendedTextMessage?.text
    if (!text) return

    // 🔹 all command
    if (text.toLowerCase() === 'all') {
      const groupMetadata = await sock.groupMetadata(from)
      const participants = groupMetadata.participants
      const mentions = participants.map(p => p.id)
      const names = participants.map(p => '@' + p.id.split('@')[0]).join(' ')
      await sock.sendMessage(from, { text: names, mentions })
    }

    // 🔹 bot reply
    if (text.toLowerCase().includes('bot')) {
      await sock.sendMessage(from, { text: 'Rabbi tor jamai 😎' })
    }
  })

  // 🔹 group updates
  sock.ev.on('group-participants.update', async (update) => {
    const { id, participants, action } = update
    for (let user of participants) {
      if (action === 'add')
        await sock.sendMessage(id, { text: `Welcome amar bow 💖 @${user.split('@')[0]}`, mentions: [user] })
      else if (action === 'remove')
        await sock.sendMessage(id, { text: `Well miss 😔 @${user.split('@')[0]}`, mentions: [user] })
      else if (action === 'promote')
        await sock.sendMessage(id, { text: `Welcome new admin 😍 @${user.split('@')[0]}`, mentions: [user] })
      else if (action === 'demote')
        await sock.sendMessage(id, { text: `Bad luck 😢 @${user.split('@')[0]}`, mentions: [user] })
    }
  })
}

startBot()
