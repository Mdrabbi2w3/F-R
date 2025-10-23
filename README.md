# F-R
Fully-featured WhatsApp bot
my-whatsapp-bot/
 ├─ index.js
 ├─ config.json
 ├─ package.json
 ├─ commands/
 │    ├─ all.js
 │    └─ menu.js
 └─ events/
      ├─ welcome.js
      └─ leave.js
     {
  "ownerNumber": "YOUR_PHONE_WITH_COUNTRYCODE",
  "prefix": "!",
  "welcomeMessage": "Welcome @user 👋",
  "goodbyeMessage": "We’ll miss you @user 😢"
}module.exports = {
  name: 'all',
  description: 'Mention all group members',
  execute: async (client, message, groupMembers) => {
    let text = groupMembers.map(m => '@' + m.id).join(' ');
    client.sendMessage(message.from, text, { mentions: groupMembers });
  }
}module.exports = {
  name: 'menu',
  description: 'Show bot commands',
  execute: async (client, message) => {
    let text = `Available commands:\n!all - Mention all\n.menu - Show this menu`;
    client.sendMessage(message.from, text);
  }
}module.exports = (client, update) => {
  if(update.action === 'add'){
    client.sendMessage(update.id, `Welcome @${update.participant}`, { mentions: [update.participant] });
  }
}module.exports = (client, update) => {
  if(update.action === 'remove'){
    client.sendMessage(update.id, `We’ll miss you @${update.participant}`, { mentions: [update.participant] });
  }
}const { WAConnection } = require('@adiwajshing/baileys');
const fs = require('fs');
const config = require('./config.json');

const client = new WAConnection();

(async () => {
  await client.connect();
  console.log('Bot connected!');

  // Load commands
  client.commands = new Map();
  const commandFiles = fs.readdirSync('./commands').filter(f => f.endsWith('.js'));
  for(const file of commandFiles){
    const command = require(`./commands/${file}`);
    client.commands.set(command.name, command);
  }

  // Load events
  const eventFiles = fs.readdirSync('./events').filter(f => f.endsWith('.js'));
  for(const file of eventFiles){
    const event = require(`./events/${file}`);
    client.on('group-participants-update', (update) => event(client, update));
  }

  // Listen messages
  client.on('chat-update', async (chat) => {
    if(!chat.hasNewMessage) return;
    const message = chat.messages.all()[0];
    if(!message.message) return;

    const text = message.message.conversation || '';
    const args = text.split(' ');
    const commandName = args[0].slice(config.prefix.length);

    if(client.commands.has(commandName)){
      const command = client.commands.get(commandName);
      const groupMembers = message.key.remoteJid.participants || [];
      command.execute(client, message, groupMembers);
    }
  });
})();
cd path/to/my-whatsapp-bot
npm install
node index.js
