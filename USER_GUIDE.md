# Helix ALM Webhook Receiver - User Guide

## What is this?

This is a **webhook receiver** - a small program that sits on your computer and waits to receive notifications (called "webhooks") from Helix ALM. Think of it like a mailbox that receives special messages from Helix ALM when certain events happen.

**In simple terms:** When something happens in Helix ALM (like a test case is updated), Helix ALM can send a notification to this program running on your computer. This program receives that notification and can log it or process it.

## What You'll Need

Before you start, make sure you have:

1. **Node.js installed** on your computer
   - If you don't have it, download it from: https://nodejs.org/
   - Choose the "LTS" (Long Term Support) version
   - After installing, you can verify it worked by opening a terminal and typing: `node --version`
   - You should see a version number (like v18.17.0)

2. **A terminal/command prompt**

## Step-by-Step Setup Instructions

### Step 1: Open Terminal

Open a fresh terminal after installing Node.

### Step 2: Navigate to the Project Folder

In the Terminal, type this command (replace the path with where you actually saved the folder):

```bash
cd [PATH TO ROOT OF PROJECT]
```

### Step 3: Install Required Packages

Type this command and press Enter:

```bash
npm install
```

This will download all the necessary code libraries. It might take a minute or two. You'll see a lot of text scrolling by - that's normal! When it's done, you'll see your command prompt again.

### Step 4: Create SSL Certificates (Required for HTTPS)

This program uses HTTPS (secure connection), so you need to create security certificates. Don't worry - these are just for local testing on your computer.

In the Terminal, run this command:

```bash
openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 365 -nodes -subj "/CN=localhost"
```

Press Enter. You may be prompted to enter a passphrase - just press Enter to skip it (it's safe for local testing). When it's done, you should see two new files created: `key.pem` and `cert.pem`.

**Note:** If you see a warning about "localhost" being insecure, that's fine - we're only using this on your local computer for testing.

### Step 5: Start the Webhook Receiver

Now you're ready to start the program! Type:

```bash
npm run start
```

Press Enter. 

**What to expect:**
- The program will start running
- You won't see much output unless you've configured it to show messages
- The program is now "listening" and waiting for webhooks
- **Don't close the Terminal window** - if you do, the program will stop

You should see the Terminal just sitting there, waiting. That means it's working!

## How to Use It

### Understanding What's Happening

1. **The program is running** - It's waiting on port 3000 for webhook notifications
2. **It's listening at:** `https://localhost:3000/`
3. **It only accepts POST requests** - This means you can't just visit it in a web browser
4. **It's designed to receive webhooks from Helix ALM** - When Helix ALM sends a webhook, this program will receive it

### What You'll See

- **If console output is enabled:** You'll see webhook details printed in the Terminal when they arrive
- **If file output is enabled:** Webhooks will be saved to a file called `receivedWebhooks.txt`
- **By default:** The program runs silently - it receives webhooks but doesn't show them unless configured

### Testing That It Works

To test if the program is running, you can send a test webhook using this command in a **new Terminal window** (keep the first one running):

```bash
curl -X POST https://localhost:3000/ -k -H "Content-Type: application/json" -d '{"test":"message"}'
```

**What this does:**
- Sends a fake webhook to your receiver
- The `-k` flag tells curl to ignore SSL certificate warnings (safe for local testing)
- If it works, you'll see nothing (or a status code) - that's normal!

## How to Stop the Program

When you're done:

1. Go back to the Terminal window where the program is running
2. Press `Ctrl + C` (hold Control and press C)
3. The program will stop and you'll see your command prompt again

### Configuring Webhook Signatures (Optional)

If Helix ALM is configured to send signed webhooks:

1. Open the file `sharedSecretKey.txt` in a text editor
2. Enter your shared secret key in this format: `sha256:your_secret_key_here`
   - Replace `sha256` with `sha512` if that's what Helix ALM uses
   - Replace `your_secret_key_here` with your actual secret key
3. Save the file
4. Stop the program and restart the program using the steps above.

The program will automatically validate webhook signatures using this key.

## Troubleshooting

### Problem: "Cannot find module" error

**Solution:** Make sure you ran `npm install` in Step 3. If you did, try running it again.

### Problem: "EADDRINUSE: address already in use :::3000"

**Solution:** Port 3000 is already being used by another program. Either:
- Stop the other program using port 3000, OR
- Find what's using it: `lsof -ti:3000` then kill it: `kill -9 [number that appeared]`

### Problem: "Cannot find module './key.pem'" or certificate errors

**Solution:** You need to create the SSL certificates. Go back to Step 4 and create them.

### Problem: "Command not found: npm" or "Command not found: node"

**Solution:** Node.js isn't installed or isn't in your PATH. 
- Install Node.js from https://nodejs.org/
- After installing, close and reopen Terminal
- Try again

### Problem: The program starts but nothing happens

**Solution:** This is actually normal! The program is waiting for webhooks. It won't do anything until Helix ALM (or you) sends a webhook to it. The program is working correctly if it's just sitting there waiting.

### Problem: Browser shows "This site can't be reached" or certificate warnings

**Solution:** This is expected! This program doesn't serve web pages - it only receives webhook POST requests. You can't visit it in a browser. It's working correctly.

## Configuration Options

You can modify `app.js` to change how the program behaves:

```javascript
await webhookReceiver.setup(3000, 200, true, true);
```

The numbers mean:
- **3000** = Port number (change if 3000 is already in use)
- **200** = Status code to return (200 = success)
- **true** = Show output in console (true) or hide it (false)
- **true** = Save webhooks to file (true) or don't save (false)

## What Happens When a Webhook Arrives?

1. Helix ALM sends a POST request to `https://localhost:3000/`
2. The program receives it
3. If signature validation is configured, it checks the signature
4. It returns a status code (default: 200) to Helix ALM
5. If console output is enabled, it prints the webhook details
6. If file output is enabled, it saves to `receivedWebhooks.txt`
7. The webhook data is stored in memory

## Getting Help

If you're stuck:
1. Check that all steps were completed correctly
2. Make sure Node.js is installed and up to date
3. Verify the SSL certificates exist (key.pem and cert.pem files)
4. Check that no other program is using port 3000
5. Try stopping and restarting the program

## Summary

- **What it does:** Receives webhook notifications from an ALM
- **How to start:** Run `npm run start` in Terminal
- **Where it listens:** `https://localhost:3000/`
- **How to stop:** Press `Ctrl + C` in Terminal
- **What to expect:** The program sits and waits for webhooks

Good luck! 🚀

