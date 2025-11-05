// ===========================
// ✅ Keep-alive Web Server
// ===========================
import express from "express";
const app = express();

app.get("/", (req, res) => {
  res.send("WTK Minecraft Indicator is alive!");
});

app.listen(3000, () => console.log("✅ Keep-alive web server running on port 3000."));

// ===========================
// 🎮 Discord + Minecraft Bot
// ===========================
import {
  Client,
  GatewayIntentBits,
  ActionRowBuilder,
  ButtonBuilder,
  ButtonStyle
} from "discord.js";
import { status } from "minecraft-server-util";
import dotenv from "dotenv";

dotenv.config();

const requiredEnvVars = ["BOT_TOKEN", "CHANNEL_ID", "SERVER_IP", "REDIRECT_URL"];
const missingVars = requiredEnvVars.filter((varName) => !process.env[varName]);

if (missingVars.length > 0) {
  console.error("❌ Missing required environment variables:", missingVars.join(", "));
  console.error("Please set these in your Replit Secrets or .env file");
  process.exit(1);
}

const client = new Client({ intents: [GatewayIntentBits.Guilds, GatewayIntentBits.GuildMessages] });

const TOKEN = process.env.BOT_TOKEN;
const CHANNEL_ID = process.env.CHANNEL_ID;
const SERVER_IP = process.env.SERVER_IP;
const SERVER_PORT = parseInt(process.env.SERVER_PORT) || 25565;
const REDIRECT_URL = process.env.REDIRECT_URL;

const BOT_NAME = "King Ciel";
const BOT_AVATAR = "https://media.discordapp.net/attachments/1067822534311039036/1435222302106320936/images.png";

let serverStartTime = null;
let messageId = null;
let cachedChannel = null;
let isOnline = false;
let players = "0/0";
let lastState = null;

function formatDuration(ms) {
  const totalSeconds = Math.floor(ms / 1000);
  const hours = String(Math.floor(totalSeconds / 3600)).padStart(2, "0");
  const minutes = String(Math.floor((totalSeconds % 3600) / 60)).padStart(2, "0");
  const seconds = String(totalSeconds % 60).padStart(2, "0");
  return `${hours}:${minutes}:${seconds}`;
}

async function updateServerStatus() {
  try {
    const res = await status(SERVER_IP, SERVER_PORT);
    if (!serverStartTime) serverStartTime = new Date();
    isOnline = true;
    players = `${res.players.online}/${res.players.max}`;
  } catch (error) {
    console.error(`Failed to connect to Minecraft server ${SERVER_IP}:${SERVER_PORT}:`, error.message);
    isOnline = false;
    players = "0/0";
    serverStartTime = null;
  }
}

async function updateDiscordMessage() {
  try {
    if (!cachedChannel) {
      cachedChannel = await client.channels.fetch(CHANNEL_ID);
    }

    const currentState = `${isOnline}-${players}`;
    const color = isOnline ? 0x1E3A8A : 0x111111;
    const runtimeDisplay = isOnline ? formatDuration(new Date() - serverStartTime) : "00:00:00";

    const row = new ActionRowBuilder().addComponents(
      new ButtonBuilder().setLabel("🎮 Join Server").setStyle(ButtonStyle.Link).setURL(REDIRECT_URL),
      new ButtonBuilder().setLabel("📋 Copy IP").setStyle(ButtonStyle.Link).setURL(REDIRECT_URL)
    );

    const embed = {
      title: "✨ WTK Minecraft Server ✨",
      description: `\`\`\`
─────────────────────────────
Status  : ${isOnline ? "🟢 Online" : "🔴 Offline"}
Players : 👥 ${players}
Runtime : ⏱️ ${runtimeDisplay}
IP      : 🖧 ${SERVER_IP}:${SERVER_PORT}
─────────────────────────────
\`\`\``,
      color: color,
      footer: { text: "🏰 King Ciel • Phantomhive" },
      timestamp: new Date()
    };

    if (messageId) {
      try {
        const msg = await cachedChannel.messages.fetch(messageId);
        await msg.edit({ embeds: [embed], components: [row] });
      } catch (error) {
        console.error("Error editing message, will create new one:", error.message);
        messageId = null;
      }
    }

    if (!messageId) {
      const msg = await cachedChannel.send({
        embeds: [embed],
        components: [row]
      });
      messageId = msg.id;
      console.log(`✅ Status message created with ID: ${messageId}`);
    }

    lastState = currentState;
  } catch (error) {
    console.error("Error updating Discord message:", error.message);
  }
}

client.once("ready", async () => {
  console.log(`✅ Logged in as ${client.user.tag}`);

  await updateServerStatus();
  await updateDiscordMessage();

  setInterval(updateServerStatus, 1000);
  setInterval(updateDiscordMessage, 1000);
});

client.on("error", (error) => {
  console.error("Discord client error:", error);
});

client
  .login(TOKEN)
  .catch((error) => {
    console.error("Failed to login to Discord:", error.message);
    console.error("Please make sure BOT_TOKEN is set correctly in your environment variables.");
  });
