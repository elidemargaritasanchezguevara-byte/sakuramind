# sakuramind
Mi app educativa con IA
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Chat IA</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      max-width: 700px;
      margin: 40px auto;
      padding: 20px;
    }
    #chat {
      border: 1px solid #ccc;
      padding: 15px;
      min-height: 300px;
      margin-bottom: 10px;
      overflow-y: auto;
    }
    .user { color: blue; }
    .bot { color: green; }
    input {
      width: 75%;
      padding: 10px;
      font-size: 16px;
    }
    button {
      width: 20%;
      padding: 10px;
      font-size: 16px;
    }
  </style>
</head>
<body>
  <h1>Mi chat con IA</h1>

  <div id="chat"></div>

  <input id="message" placeholder="Escribí tu mensaje..." />
  <button onclick="sendMessage()">Enviar</button>

  <script>
    const chat = document.getElementById("chat");
    const messageInput = document.getElementById("message");

    function addMessage(text, className) {
      const p = document.createElement("p");
      p.className = className;
      p.textContent = text;
      chat.appendChild(p);
    }

    async function sendMessage() {
      const message = messageInput.value.trim();
      if (!message) return;

      addMessage("Vos: " + message, "user");
      messageInput.value = "";

      const res = await fetch("/.netlify/functions/chat", {
        method: "POST",
        headers: {
          "Content-Type": "application/json"
        },
        body: JSON.stringify({ message })
      });

      const data = await res.json();

      if (data.reply) {
        addMessage("IA: " + data.reply, "bot");
      } else {
        addMessage("Error: " + (data.error || "No respondió"), "bot");
      }
    }

    messageInput.addEventListener("keydown", (e) => {
      if (e.key === "Enter") sendMessage();
    });
  </script>
</body>
</html> {
  "name": "mi-chat",
  "version": "1.0.0",
  "private": true,
  "dependencies": {
    "openai": "^4.0.0"
  }
}[build]
  publish = "."
  functions = "netlify/functions"const OpenAI = require("openai");

const client = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY,
});

exports.handler = async function (event) {
  try {
    const body = JSON.parse(event.body || "{}");
    const message = body.message;

    if (!message) {
      return {
        statusCode: 400,
        body: JSON.stringify({ error: "Falta el mensaje" }),
      };
    }

    const response = await client.responses.create({
      model: "gpt-4.1-mini",
      input: message,
    });

    return {
      statusCode: 200,
      body: JSON.stringify({
        reply: response.output_text,
      }),
    };
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({ error: "Error del servidor" }),
    };
  }
};