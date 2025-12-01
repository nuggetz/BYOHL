# Deploying On-Premises Connector on your machine with Docker

🚀 This guide will walk you through the process of deploying an on-premises connector on your server using Docker. The on-premises connector allows secure communication between your server and remote resources through Twingate.

## Prerequisites

📋 A machine running Linux Server 22.4 or +
☑️ Docker engine installed

## Step 1: Twingate Admin Account Creation

1. Visit [Twingate's website](https://www.twingate.com/) and click on "**Pricing**" 🌐
2. Under the "**Free**" tier, click on "**Get Started**" 🆓
3. Sign up with your favorite SSO or with your email ✉️
4. You will be prompted to enter a name for your "**Company**". If you are using it for personal use, just name it whatever you want 🏢
5. Now select "**Yes, I'm able to run CLI commands**" and agree ✅
6. You have to choose the **URL** for the Admin Dashboard (it could be the same as your company name) 🖥️

🎉 Congratulations, you now have a Twingate Account.

## Step 2: Define your Remote Network

1. Click on the "**Add your Remote Network**" button on the right side of the screen ➕
2. Choose "**On-Premises**" 🏢
3. Enter the name of the remote network and confirm ✔️

## Step 3: Deploy a Connector

1. On the page of your newly created remote network, there are two "**Deploy Connector**" buttons. Choose one and click it ⚙️
2. Now choose to deploy the connector using Docker 🐳
3. Click on "**Generate Tokens**" and re-authenticate to your account 🔒
4. With this done, skip to step number four and copy the CLI command 📋
5. On your machine, paste the CLI Command (remember to use SSH for a more convenient procedure) 🍓

🎉 Congratulations, you have deployed a connector! You can verify it by the green LED near the connector in the Twingate admin page.

## Step 3: Add a Resource to your Remote Network

1. On the main page of your newly created Network, click on the "**+**" button to add a resource ➕
2. Select your remote network from the drop-down menu ⬇️
3. Select "**CIDR**" and name your resource in the "**Label**" field 🏷️
4. Insert your machine's IP Address in the "**CIDR Address**" field 🖥️
5. Click on save ✔️
6. Finally, you will be prompted to choose any user restrictions if you have set up user groups. If not, just select "**Everyone**" 👥

## Step 4: Downloading the Client and Trying the Connection

1. Visit Twingate's [Downloads Page](https://www.twingate.com/download) 🌐
2. Choose your operating system 💻
3. Download the client ⬇️
4. Install it ⚙️
5. Authenticate with your Twingate credentials 🔒
6. If you have a mobile phone, you can download the client from the iOS or Google Play store 📱

🎉 Congratulations! You are now able to reach your Home Assistant anywhere you are.
