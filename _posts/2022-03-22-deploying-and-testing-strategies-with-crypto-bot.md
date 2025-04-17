---
title: "Deploying a Crypto Trading Bot to Test Strategies on Cryptocoins"
excerpt_separator: "<!--more-->"
categories:
  - Blog
tags:
  - Crypto
  - Bitcoin
  - Quant
comments: true
---

Automated crypto trading can save time, eliminate emotions, and optimize your strategies. In this guide, we’ll walk through setting up a fully functional trading bot using [Freqtrade](https://www.freqtrade.io/en/stable/)---an open-source crypto trading framework---and deploy it on the AWS cloud.

## Getting Started: Building the Bot

To get your bot up and running, you can follow either of these resources:

- The official [Freqtrade Documentation](https://www.freqtrade.io/en/stable/)
- This practical [Udemy Course](https://www.udemy.com/course/build-a-crypto-bot-100-functional-algorithmic-trading-freqtrade/) that walks you through building a bot from scratch

## Deploying on AWS Cloud

Here’s a step-by-step guide to deploy your bot using AWS EC2:

1. **Create an AWS Account**  
   Head over to [AWS](https://aws.amazon.com/) and register.

2. **Launch a Debian-Based EC2 Instance**  
   Choose a lightweight Debian-based Linux AMI and generate a key pair (`.pem` file).

3. **Connect to the EC2 Instance**  
   Use your terminal to SSH into your instance:
   ```bash
   ssh -i "KeyPair.pem" admin@<your-ec2-ip>
   ```

4. **Install Freqtrade**  
   Follow the installation steps from the [Freqtrade Installation Guide](https://www.freqtrade.io/en/stable/installation/).  
   > Tip: Make sure you meet all the system requirements before proceeding.

5. **Activate the Virtual Environment**  
   Once installed, activate the Python virtual environment:
   ```bash
   source ./.env/bin/activate
   ```

6. **Configure the Bot**  
   Set up your bot by generating a new config file:
   ```bash
   freqtrade new-config
   ```
   This will create a `config.json` where you'll add your exchange and trading parameters.

7. **Connect to a Crypto Exchange**  
   - Generate your **API key** and **secret** from an exchange like **Binance** or **Kraken**.
   - Input these into your `config.json` to enable live trading or paper trading.

8. **Telegram Integration (Optional but Recommended)**  
   - Create a Telegram bot and get its token.
   - Find your Telegram chat ID.
   - Add both to the config file to monitor and control the bot via Telegram.

9. **Keep the Bot Running After Disconnection**  
   Use `screen` to keep the bot running even if you close your terminal:
   ```bash
   screen
   ```
   Start your bot, then detach with:
   ```bash
   Ctrl + A, then D
   ```
   To resume:
   ```bash
   screen -r
   ```
   More info here: [Keep SSH Sessions Alive](https://www.tecmint.com/keep-remote-ssh-sessions-running-after-disconnection/)

## Limitations & Considerations

- **No Short Selling:** Freqtrade only supports buying/selling spot pairs, not margin or futures.
- **No Market-Neutral Strategies:** Since you can't short or access perpetual contracts, strategies like arbitrage or funding rate exploitation aren't feasible.
- **Market Risk:** Automated doesn't mean risk-free. Backtest thoroughly before going live.

## Final Thoughts

Freqtrade is a powerful tool for algorithmic crypto trading, especially for those with some coding experience. Combined with AWS, it offers a scalable and low-cost setup for your bot — whether you're just experimenting or looking to automate a production-level strategy.

Have you built your bot already? What strategy are you running? Share in the comments!
