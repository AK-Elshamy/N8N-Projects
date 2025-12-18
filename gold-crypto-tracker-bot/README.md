# 🤖 AI Financial Agent (Gold & Crypto Tracker)

An intelligent AI Agent built using **n8n** that monitors real-time market data for Gold and Cryptocurrencies. It interacts via **Telegram** and sends urgent alerts through **Pushover**.

## 🌟 Features

- **Real-time Price Fetching:** Gets current prices for Gold (21k, 24k) and Crypto (BTC, ETH, SOL).
- **Currency Conversion:** Automatically converts USD prices to EGP.
- **Time-Aware:** Uses a custom tool to fetch the current local time for accurate reporting.
- **Smart Notifications:** Sends regular updates to Telegram and high-priority alerts to Pushover.
- **Interactive Chat:** Capable of introducing itself and explaining its financial services.

## 🛠️ Tech Stack

- **n8n:** Workflow automation and AI Agent orchestration.
- **OpenAI (GPT-4o):** The brain of the agent.
- **Telegram Bot API:** For user interaction.
- **Pushover API:** For urgent mobile notifications.
- **GoldAPI.io:** Real-time gold market data.
- **CoinGecko API:** Cryptocurrency market data.

## ⚙️ Setup & Tools

To run this agent, you will need to configure the following tools in n8n:

1. **Telegram Tool:** To send/receive messages.
2. **Pushover Tool:** For critical alerts.
3. **Gold Price Tool:** Using HTTP Request to `goldapi.io`.
4. **Crypto Price Tool:** Using HTTP Request to `coingecko.com`.
5. **Current Time Tool:** A custom JavaScript snippet for local time.

## 🚀 How to Use

1. Import the `.json` workflow file into your n8n instance.
2. Set up your credentials for Telegram, Pushover, and GoldAPI.
3. Activate the workflow.
4. Send `/start` or ask "بكام سعر الذهب والبيتكوين؟" to your Telegram bot.

## 📝 License
