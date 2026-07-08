# LM Studio Setup Guide (Prerequisite Environment)

> This guide isn't specific to Rashinban for AI — it covers the general steps for setting up LM Studio as your "run AI locally" environment. Complete this before moving on to Rashinban for AI's own setup.

## What Is LM Studio?

LM Studio is an app for running AI models on your own computer. Once you've downloaded a model, you can often use it without an internet connection (though downloading the model itself does require one). Rashinban for AI is designed to work alongside LM Studio (or a similar local AI environment).

## Step 1: Install LM Studio

1. Go to [https://lmstudio.ai](https://lmstudio.ai)
2. Download the installer for your OS (Windows / Mac / Linux)
3. Run the downloaded file and complete installation

## Step 2: Download an AI model

1. Open LM Studio
2. Open the search panel (magnifying glass icon) on the left
3. Search for and download a model

> If you're not sure which model to pick, start with something lightweight just to confirm everything works. How large a model you can run depends on your computer's specs (mainly RAM). If you're unsure, the recommended models on LM Studio's home screen are a good starting point.

## Step 3: Load the model

1. Select the model you downloaded and click "Load"
2. Once loading finishes, you'll be able to chat with that model in the chat screen

At this point, you have a working "I can chat with an AI model" setup.

## Step 4: Confirm Developer mode (Developer tab)

Using an MCP server (including Rashinban for AI) with LM Studio requires the Developer mode.

1. Look for a `</>`-style icon near the top or side of the screen — that's the Developer tab
2. Click to open it

If you can open the Developer tab, this guide's preparation is complete.

> ⚠️ LM Studio's layout can change between versions. If you can't find the `</>` icon, check the settings menu, or refer to LM Studio's official documentation.

## Once You're Done Here

Continue with `AI_Setup_Assistant_Prompt_EN.md`, under "Setup Steps → Step 1: Load the Chrome extension." Registering the MCP server config file (`mcp.json`) with the Developer environment you just confirmed is covered in that file's "Step 5."
