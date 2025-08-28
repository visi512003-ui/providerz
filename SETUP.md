# Setup Instructions

## Option 1: Local Development

1. Extract all files to a directory
2. Install Python 3.8+ 
3. Run: `pip install -r requirements.txt`
4. Run: `python main.py`
5. Open browser to: http://localhost:5000

## Option 2: Deploy on Replit

1. Upload the ZIP file to Replit
2. Extract files in your Repl
3. Click the Run button
4. Your app will be live instantly!

## Option 3: Deploy on Other Platforms

### Heroku:
1. Create new Heroku app
2. Upload files
3. Add Procfile: `web: python main.py`
4. Deploy

### Railway/Render:
1. Connect your GitHub repo
2. Set build command: `pip install -r requirements.txt` 
3. Set start command: `python main.py`

The app is designed to work on any platform that supports Python and Flask.
