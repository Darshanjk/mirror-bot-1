# What is this repo about?
This is a telegram bot writen in python for mirroring files on the internet to our beloved Google Drive.

[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/Darshanjk/mirror-bot-1)

# Inspiration 
This project is heavily inspired from @out386 's telegram bot which is written in JS.

# Features supported:
- Mirroring direct download links to google drive
- Download progress
- Upload progress
- Download/upload speeds and ETAs
- Docker support
- Uploading To Team Drives.
- Index Link support

# Upcoming features (TODOs):
- Mirror from Telegram files

# How to deploy?
Deploying is pretty much straight forward and is divided into several steps as follows:
## Installing requirements

- Clone this repo:
```
git clone https://github.com/lzzy12/python-aria-mirror-bot mirror-bot/
cd mirror-bot
```

- Install requirements
For Debian based distros
```
sudo apt install python3
sudo snap install docker 
```
- For Arch and it's derivatives:
```
sudo pacman -S docker python
```

## Setting up config file
```
cp config_sample.env config.env
```
- Remove the first line saying:
```
_____REMOVE_THIS_LINE_____=True
```
Fill up rest of the fields. Meaning of each fields are discussed below:
- BOT_TOKEN : The telegram bot token that you get from @BotFather
- GDRIVE_FOLDER_ID : This is the folder ID of the Google Drive Folder to which you want to upload all the mirrors.
- DOWNLOAD_DIR : The path to the local folder where the downloads should be downloaded to
- DOWNLOAD_STATUS_UPDATE_INTERVAL : A short interval of time in seconds after which the Mirror progress message is updated. (I recommend to keep it 5 seconds at least)  
- OWNER_ID : The Telegram user ID (not username) of the owner of the bot
- AUTO_DELETE_MESSAGE_DURATION : Interval of time (in seconds), after which the bot deletes it's message (and command message) which is expected to be viewed instantly. Note: Set to -1 to never automatically delete messages
- IS_TEAM_DRIVE : (Optional field) Set to "True" if GDRIVE_FOLDER_ID is from a Team Drive else False or Leave it empty. 
- INDEX_URL : (Optional field) Refer to https://github.com/maple3142/GDIndex/ The URL should not have any trailing '/'

Note: You can limit maximum concurrent downloads by changing the value of MAX_CONCURRENT_DOWNLOADS in aria.sh. By default, it's set to 2
 
## Getting Google OAuth API credential file

- Visit the Google Cloud Console
- Go to the OAuth Consent tab, fill it, and save.
- Go to the Credentials tab and click Create Credentials -> OAuth Client ID
- Choose Other and Create.
- Use the download button to download your credentials.
- Move that file to the root of mirror-bot, and rename it to credentials.json
- Finally, run the script to generate token file (token.pickle) for Google Drive:
```
pip install google-api-python-client google-auth-httplib2 google-auth-oauthlib
python3 generate_drive_token.py
```
## Deploying

- Start docker daemon (skip if already running):
```
sudo dockerd
```
- Build Docker image:
```
sudo docker build . -t mirror-bot
```
- Run the image:
```
sudo docker run mirror-bot
```

## Using service accounts for uploading to avoid user rate limit

Many thanks to [AutoRClone](https://github.com/xyou365/AutoRclone) for the scripts
### Generating service accounts
Step 1. Generate service accounts [What is service account](https://cloud.google.com/iam/docs/service-accounts) [How to use service account in rclone](https://rclone.org/drive/#service-account-support).
---------------------------------
Let us create only the service accounts that we need. 
**Warning:** abuse of this feature is not the aim of autorclone and we do **NOT** recommend that you make a lot of projects, just one project and 100 sa allow you plenty of use, its also possible that overabuse might get your projects banned by google. 

```
Note: 1 service account can copy around 750gb a day, 1 project makes 100 service accounts so thats 75tb a day, for most users this should easily suffice. 
```

`python3 gen_sa_accounts.py --quick-setup 1 --new-only`

A folder named accounts will be created which will contain keys for the service accounts created
```
We highly recommend to zip this folder and store it somewhere safe, so that you do not have to create a new project everytime you want to deploy the bot
```
### Adding service accounts to Google Groups:
We use Google Groups to manager our service accounts considering the  
[Official limits to the members of Team Drive](https://support.google.com/a/answer/7338880?hl=en) (Limit for individuals and groups directly added as members: 600).

1. Turn on the Directory API following [official steps](https://developers.google.com/admin-sdk/directory/v1/quickstart/python) (save the generated json file to folder `credentials`).

2. Create group for your organization [in the Admin console](https://support.google.com/a/answer/33343?hl=en). After create a group, you will have an address for example`sa@yourdomain.com`.

3. Run `python3 add_to_google_group.py -g sa@yourdomain.com`

4. Now, add Google Groups (**Step 2**) to manager your service accounts, add the group address `sa@yourdomain.com` or `sa@googlegroups.com` to the Team drive or folder
