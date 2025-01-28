# MAPXOR1210

```js
const { Client, Intents } = require("discord.js-selfbot-v13");
const client = new Client({ checkUpdate: false }); // Disable update checks for selfbots

// Login with your token
const token = "YOUR_DISCORD_TOKEN"; // Replace with your token
client.login(token);

client.on("ready", () => {
  console.log(`Logged in as ${client.user.tag}`);
});

client.on("messageCreate", async (message) => {
  // Check if the message starts with !check and has a valid message link
  if (message.content.startsWith("!check")) {
    const args = message.content.split(" ");
    if (args.length < 2) {
      message.channel.send("Please provide a valid message link.");
      return;
    }

    const messageLink = args[1];
    const match = messageLink.match(
      /https:\/\/discord\.com\/channels\/(\d+)\/(\d+)\/(\d+)/
    );

    if (!match) {
      message.channel.send("Invalid message link format.");
      return;
    }

    const [_, guildId, channelId, messageId] = match;

    try {
      const guild = client.guilds.cache.get(guildId);
      const channel = guild.channels.cache.get(channelId);
      const targetMessage = await channel.messages.fetch(messageId);

      const reactions = targetMessage.reactions.cache;
      const allReactors = [];

      // Fetch all reactions and users
      for (const reaction of reactions.values()) {
        const users = await reaction.users.fetch();
        users.forEach((user) => {
          if (!allReactors.includes(user.id)) {
            allReactors.push(user.id);
          }
        });
      }

      // Check if the number of reactions exceeds 300
      if (allReactors.length > 300) {
        message.channel.send(
          `The message has more than 300 unique reactions (${allReactors.length}).`
        );
      }

      // Send p <user.id> messages with a 21-second timeout
      for (const userId of allReactors) {
        await message.channel.send(`p <${userId}>`);
        console.log(`Sent p <${userId}>`);
        await new Promise((resolve) => setTimeout(resolve, 21000)); // Wait 21 seconds
      }
    } catch (error) {
      console.error(error);
      message.channel.send("An error occurred while fetching the message.");
    }
  }
});
```
