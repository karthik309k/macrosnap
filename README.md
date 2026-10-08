# 🥗 MacroSnap

**Snap it. Track it. Text yourself the results.**

MacroSnap is an AI nutrition buddy built with Streamlit. Chat with it by typing what you ate or attaching a photo of your meal, and get an instant calorie and macro estimate. When you're done, one button sends a full summary of your conversation straight to your WhatsApp.

Built with **Google Gemini** (chat + vision) and **Twilio** (WhatsApp). No OpenCV, no MediaPipe, no model training.

**Live demo:** https://macrosnap-v4kz.onrender.com/


## Features

- 💬 Chat with an AI nutrition buddy that remembers the whole conversation
- 📸 Photo understanding: attach a meal photo and get calories and macros
- 🎯 Stays on topic: politely declines anything unrelated to food or fitness
- 📲 One-click WhatsApp summary of every meal you logged, with running totals
- 🔁 Automatic retry when Gemini is temporarily busy (503 errors)

## Tech stack

| Part | Tool |
|---|---|
| UI | Streamlit |
| AI (chat + vision) | Google Gemini via google-genai |
| Messaging | Twilio WhatsApp (Content Templates) |
| Hosting | Render / Streamlit Community Cloud |

## Project structure

macrosnap/
├── app.py                          # the app itself
├── prompts.py                      # the AI's personality, kept separate
├── requirements.txt                # dependencies
├── .gitignore                      # keeps secrets.toml out of GitHub
└── .streamlit/
    └── secrets.toml.example        # template: copy to secrets.toml and fill in

## Prerequisites

- Python 3.9 or newer
- A free [Google AI Studio](https://aistudio.google.com) account (Gemini API key)
- A free [Twilio](https://www.twilio.com/try-twilio) account (WhatsApp sandbox)

## Setup

### 1. Clone and install

bash
git clone https://github.com/karthik309k/macrosnap.git
cd macrosnap
python -m venv venv


Activate the virtual environment:

- **macOS / Linux:** `source venv/bin/activate`
- **Windows PowerShell:** `.\venv\Scripts\Activate.ps1`
- **Windows cmd:** `venv\Scripts\activate.bat`

Then install the dependencies:

bash
pip install -r requirements.txt

### 2. Get your keys

1. **Gemini:** create an API key at [aistudio.google.com](https://aistudio.google.com).
2. **Twilio:** from the [Twilio Console](https://console.twilio.com), copy your **Account SID** and **Auth Token**.
3. **WhatsApp sandbox:** go to *Messaging → Try it out → Send a WhatsApp message*. From your phone, send the join message (for example `join happy-tiger`) to the sandbox number. This opt-in expires after about 72 hours of inactivity.
4. **Content Template:** go to *Messaging → Content Template Builder* and create a **Text** template:
   
   Hi {{1}}, here's your MacroSnap summary:

   {{2}}
   
   Copy its **Content SID** (starts with "HX...").

> **Why a Content Template?** WhatsApp only allows free-form replies within 24 hours of the customer's last message. A message the app sends on its own, like the summary button, needs an approved template.

### 3. Add your secrets

Copy .streamlit/secrets.toml.example to .streamlit/secrets.toml and fill in the values:

toml
GEMINI_API_KEY = "your-gemini-api-key"
TWILIO_ACCOUNT_SID = "ACxxxxxxxxxxxxxxxx"
TWILIO_AUTH_TOKEN = "your-twilio-auth-token"
TWILIO_WHATSAPP_FROM = "whatsapp:+14155238886"
TWILIO_CONTENT_SID = "HXxxxxxxxxxxxxxxxx"

⚠️ Never commit `secrets.toml`. It is already listed in `.gitignore`.

### 4. Run it

bash
streamlit run app.py

Open http://localhost:8501, enter your name and the WhatsApp number that joined the sandbox, then chat. After you've logged at least one meal, click **Send to WhatsApp**.

## Deployment

The app reads each secret from an environment variable first, then falls back to `secrets.toml`, so the same code runs everywhere.

### Render

1. Create a new **Web Service** from this GitHub repo.
2. Settings:
   - **Runtime:** Python 3
   - **Build command:** pip install -r requirements.txt
   - **Start command:** streamlit run app.py --server.port $PORT --server.address 0.0.0.0
3. Under **Environment**, add the five variables from the table above (without quotes).
4. Deploy.

On Render's free plan the app sleeps after about 15 minutes of inactivity, so the first visit afterwards can take 30 to 60 seconds.

### Streamlit Community Cloud

1. Go to [share.streamlit.io](https://share.streamlit.io) and sign in with GitHub.
2. Click **Create app**, pick this repo, branch main, and app.py.
3. In **Advanced settings → Secrets**, paste the contents of your local secrets.toml.
4. Deploy.

## Customizing

- **Personality and tone:** edit SYSTEM_PROMPT in prompts.py.
- **Welcome message:** edit WELCOME_MESSAGE_TEMPLATE in prompts.py.
- **Summary format:** edit SUMMARY_REQUEST_PROMPT in prompts.py.
- **Model:** change MODEL_NAME in app.py. Try gemini-3.5-flash if a newer model is busy.

## Troubleshooting

| Problem | Fix |
|---|---|
| `No secrets found` error | Make sure `.streamlit/secrets.toml` exists (not `.example`) or the environment variables are set. |
| `503 UNAVAILABLE` from Gemini | The model is busy. The app retries automatically; if it persists, switch `MODEL_NAME`. |
| WhatsApp send fails | Check that the number joined the sandbox, the Content SID is correct, and the template has exactly `{{1}}` and `{{2}}`. |
| Messages stop arriving | The sandbox join expired. Send the join message again from your phone. |
| Send button is greyed out | Log at least one meal first. |

## How it works

1. **Onboarding:** the user enters a name and WhatsApp number. A Gemini chat session is created with the system prompt and stored in Streamlit's session state.
2. **Chat:** text and photos are sent to the same chat session, so follow-up questions work.
3. **Summary:** the send button asks Gemini (behind the scenes) to summarize the whole conversation, then sends the result through Twilio's Content API.

## License

Free to use for learning and workshops.
