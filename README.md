## biomeTREEx
Overview
This sophisticated n8n workflow automatically generates and iteratively updates a "Digital Biometric Garden" image based on your daily health data from the Oura Ring. It translates metrics like sleep, readiness, and activity into elements of a virtual garden.

The key feature of this workflow is its iterative nature. Instead of generating a new image from scratch each day, it analyzes how your biometric data has changed. It uses Google Gemini to compose precise editing instructions for the Black Forest Labs (BFL) FLUX.1 Kontext image model. FLUX.1 then modifies the previous day's image to reflect the new state, creating a consistent, evolving visual narrative of your health journey.

<img width="1024" height="1024" alt="test" src="https://github.com/user-attachments/assets/aa527ba2-f384-4b9d-a84a-9b1841e2f857" />

<img width="1024" height="1024" alt="current" src="https://github.com/user-attachments/assets/14e48b25-80a0-4034-91ae-f3992e702a70" />



## Table of Contents
Features
How It Works
The Digital Garden Metaphor
Prerequisites

Setup Instructions
1. Import the Workflow
2. Configure Credentials
3. Configure Google Sheets
4. Configure Local File Access
5. Configure Telegram

Initialization: Generating the First Image (Critical Step)
Usage and Scheduling
Workflow JSON

## Features
**Oura Ring Integration:** Automatically fetches Sleep, Readiness, and Activity summaries.

**Biometric Mapping:** Translates complex health data into visual metaphors (the "Digital Garden").

**AI-Powered Prompt Engineering:** Uses Google Gemini (via LangChain) to analyze changes and generate precise image editing instructions.

**Iterative Image Generation:** Utilizes the BFL FLUX.1 Kontext model for context-aware image editing, ensuring consistency day-to-day.

**State Management:** Uses Google Sheets to store daily prompts and track historical changes.

**Notifications:** Sends the updated daily garden image via Telegram.

**Automation:** Designed to run automatically on a daily schedule.

## How It Works
**Trigger: ** The workflow is initiated either manually or via a daily schedule.

**Fetch Oura Data: **The workflow connects to the Oura API to retrieve the latest Activity, Readiness, and Sleep summaries.

**Data Interpretation** (The Digital Garden): A custom JavaScript node (Code1) processes the Oura scores. It calculates a 1-5 rating for different aspects of well-being and maps them to predefined visual descriptions of garden elements.

**State Management:**

The workflow reads the previous day's visual descriptions from a Google Sheet.

The sheet is cleared, and the new descriptions for the current day are saved.

Change Detection: The workflow (Code2) compares today's descriptions with yesterday's to identify exactly which elements of the garden need to change.

AI Prompt Composition: The identified changes are fed into a LangChain agent (Change Composer) powered by Google Gemini. This agent is instructed to write iterative editing prompts focusing only on the required modifications (e.g., "Make the stream flow faster. LEAVE ANYTHING ELSE EXACTLY LIKE IT IS NOW.")

**Image Generation (FLUX.1 Kontext):**

The workflow loads the image generated on the previous day from the n8n server's local storage.

It sends the previous image and the AI-generated editing prompt to the BFL FLUX.1 Kontext API.

The workflow polls the API until the image processing is complete.

Output: The newly generated image is downloaded, saved locally (overwriting the old image, ready for the next day's iteration), and sent to the user via Telegram.

## The Digital Garden Metaphor
The workflow uses the following mappings (rated 1-5) to translate biometric data into visual elements. The logic and descriptions are defined in the Code1 node.

Element	Domain	Description (Summary)
SKY	Sleep & Recovery	Reflects sleep quality. Ranges from stormy and overcast to perfectly clear and starry.
WATER BODY (Pond)	Cardiovascular Health & Readiness	Represents readiness and recovery (HRV, RHR). Ranges from stagnant and murky to crystal-clear and vibrant.
STREAM	Physical Energy & Activity	Represents activity score and energy. Ranges from a dry stream bed to a powerful waterfall.
TREE (Bonsai)	Mental & Emotional State	Reflects restfulness and mental load. Ranges from withered and brittle to vibrant with blossoms.
ATMOSPHERE	Overall System State	The average of all scores. Ranges from a dark thunderstorm to a perfect, clear, sunny day.

In Google Sheets exportieren
Prerequisites
To use this workflow, you will need the following:

n8n Instance: A running instance of n8n (self-hosted or cloud).

Oura Ring Account: With API access (Personal Access Token).

Google Cloud Account:

Google Sheets API: Enabled, with credentials (OAuth2 or Service Account) configured in n8n.

Google Gemini API (Vertex AI or AI Studio): An API key.

Black Forest Labs (BFL) API: An account and API key for the FLUX.1 Kontext model (https://api.bfl.ai/v1/flux-kontext-pro).

Telegram: A Bot Token and the Chat ID where you want to receive the images.

## Setup Instructions
1. Import the Workflow
Copy the workflow JSON from the Workflow JSON section below and paste it into a new workflow canvas in your n8n editor.

2. Configure Credentials
You will need to configure credentials for the following nodes:

Oura Nodes (Get activity summary, Get readiness summary, Get sleep summary): Add your Oura API credentials.

Google Sheets Nodes (Get row(s) in sheet, Clear sheet, Append row in sheet): Add your Google Sheets credentials.

Google Gemini Chat Model: Add your Google Palm/Gemini API key.

BFL API (HTTP Request Nodes): Configure httpHeaderAuth for the nodes: Start FLUX Generation Job, Do the Polling (and the initialization nodes ending in 1). This typically involves setting an Authorization header with the value Bearer YOUR_BFL_API_KEY.

Telegram Node (Send a photo message): Add your Telegram Bot Token.

3. Configure Google Sheets
Create a Google Sheet to store the daily prompts.

In the Google Sheets nodes, enter the Document ID (Spreadsheet ID) and the correct Sheet Name (e.g., gid=0). The workflow manages the data within this sheet automatically.

4. Configure Local File Access
This workflow relies heavily on reading and writing the generated image to the local disk of the n8n server for the iterative process.

Read/Write File Nodes: Locate the nodes named Read/Write Files from Disk, Read/Write Files from Disk1 (Initialization), and Read/Write Files from Disk2 (Final Output).

Specify File Path: You must specify the exact file path where the image should be stored (e.g., /home/n8n/digital_garden/today.png). All these nodes must point to the same file path.

n8n Configuration (Self-Hosted): Ensure your n8n instance is configured to allow access to this file path. If using Docker, ensure the path is correctly volume-mounted.

5. Configure Telegram
In the Send a photo message node, enter the Chat ID where the image should be sent.

Initialization: Generating the First Image (Critical Step)
Important: The iterative process requires a starting image file to exist on the disk.

The workflow includes a separate, disconnected branch (bottom left) to generate this initial image:

Locate the Start FLUX Generation Job1 node. This node contains a hardcoded, comprehensive prompt describing a default garden state.

Ensure the Read/Write Files from Disk1 node is configured to save to the correct path (see Step 4).

Run this branch manually once: Execute the Start FLUX Generation Job1 node.

The workflow will call the BFL API, poll for the result, and save the initial image to the disk.

Once this base image is saved, the main workflow is ready to run.

Usage and Scheduling
Manual Execution: You can use the When clicking ‘Execute workflow’ trigger to test the workflow.

Automatic Execution: Activate the Schedule Trigger node and set it to run once per day (e.g., every morning at 8 AM) after your Oura data has synchronized.

<img width="1024" height="1024" alt="dead" src="https://github.com/user-attachments/assets/5cf4fe00-5996-410b-9786-953c92550ab4" />

