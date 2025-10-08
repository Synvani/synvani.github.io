---
published: true
title: Deploying Python Dash Apps
author: synvan
date: 2025-02-01 14:10:00 +0200
categories: [Code, Python]
tags: [Python, Dash, Web app, PythonAnywhere]
render_with_liquid: false
description: A step-by-step guide to deploying a Python Dash app online using PythonAnywhere – from setup to live demo in minutes.
media_subpath: '/assets/img/posts/04_deploy-dash-app'
image:
  path: 00_python-dash-thumbnail.png
  alt: 
  in_post: false
---

I recently put together a small web app in Python using Dash – it was great fun to build and actually relates to a video game I was playing during peak Covid-19. The app is not too fancy, it lets you adjust many inputs and makes some calculations easier and faster, including plots for better visualization of the results.

The app worked great on my machine, but at some point I thought it was time to share it with others who might find it useful too. That's when I started looking into how to actually deploy a Dash app so it could live on the web and be accessed by anyone with a browser.

## Dash App Deployment Options

I started looking into options for hosting my Dash app online with minimal cost – it's a side-project after all with no intention to generate any profit. There are several platforms worth considering, each offers free tier or low-cost tier that get the job done. Each platform has different trade-offs in performance, resource limits, setup, and restrictions.

I did some quick research and tried to put some conclusions in a table:

| Platform           | Setup Ease                  | Free Tier Limits                                               | Best For                                |
| ------------------ | --------------------------- | -------------------------------------------------------------- | --------------------------------------- |
| **Plotly Cloud**   | Very easy (upload only)     | File size & dependency limits, fewer customization options     | Quick sharing of small Dash dashboards  |
| **Render.com**     | Very easy (GitHub + config) | Limited instance hours & bandwidth; cold starts if idle        | General-purpose hosting, Heroku-style   |
| **PythonAnywhere** | Easy (UI-based upload)      | 1 web app, 512 MB disk, CPU throttling, no custom domain       | Beginners, lightweight apps, simplicity |
| **Railway.app**    | Easy (GitHub linked)        | Small monthly credit (~$1), low RAM/CPU, ephemeral file system | Hobby projects, CI/CD experimentation   |

The choice really comes down to your requirements and what you value most. If you just want to share a simple dashboard quickly, Plotly Cloud is the fastest way. If you prefer GitHub-connected deployments and don't mind some resource limits, Render and Railway are both solid picks. Google Cloud is overkill unless you're aiming for a production-grade setup (I didn't bother adding it to the table). For my needs though PythonAnywhere was the best fit – a lightweight app, minimal setup time, and a beginner-friendly workflow.

## Step by Step with PythonAnywhere

Let's get down to business – I'll walk through all steps I did deploying my web app. A lot of the information below can be found in PythonAnywhere's documentation or support forums, but here it is all compiled and will likely work for most lightweight app configurations.

### Expose the Flask server

Before we get into PythonAnywhere's system, let's prepare the Python project first:
1. Open `app.py` (or whatever your Dash entry filename is).
2. Ensure you set `server = app.server` (PythonAnywhere needs that server object), as below:

```python
from dash import Dash, html

app = Dash(__name__)
app.layout = html.Div("Hello from Dash on PythonAnywhere!")

server = app.server   # expose the Flask server for WSGI

if __name__ == "__main__":
    app.run_server(debug=True)
```
{: file='app.py' .nolineno }


### Prepare Your Project

Make sure your project follows this structure:

```
mydashapp/
├─ app.py
├─ requirements.txt
└─ assets/
```
{: .nolineno }

Example for the contents of a `requirements.txt` file:

```
dash
dash-bootstrap-components
plotly
numpy
pandas
dash[diskcache]
```
{: file='requirements.txt' .nolineno }

   <!-- markdownlint-capture -->
   <!-- markdownlint-disable -->
   > Other deployment platforms may require you to add specific packages. For example, Render.com will instruct you to add 'gunicorn' to this `requirements.txt` file.
   {: .prompt-info }
   <!-- markdownlint-restore -->


### Start a Bash Console

1. Sign in to PythonAnywhere (or create a new account).
2. In <kbd>Dashboard</kbd> tab, under **New Console** section, find the start button to fire up a bash console. Alternatively, navigate to <kbd>Consoles</kbd> tab → start a Bash console (bottom-left of the screen).

![PythonAnywhere dashboard](10_pythonanywhere-dashboard.png){: }


### Create a Virtual Environment
1. Create a virtualenv (either `venv` or `virtualenvwrapper`, both are fine) – pick the same Python version you'll choose for the web app itself later.

    ```bash
    python3.10 -m venv ~/.virtualenvs/mydashapp
    source ~/.virtualenvs/mydashapp/bin/activate
    ```
    {: .nolineno }

    ![Creating virtual environment](20_create-venv.png){: w='500' h='500' }
    _Bash console at PythonAnywhere_

2. Clone your project from the GitHub repository (or upload files manually via the Files tab).

    ```bash
    mkdir -p ~/sites && cd ~/sites
    git clone https://github.com/<username>/<repo>.git mydashapp   
    cd mydashapp
    ```
    {: .nolineno }

    ![Cloning repository](30_clone-repo.png){: w='500' h='500' }
    _Bash console at PythonAnywhere_

3. Install all required packages (`requirements.txt`) – this may take a bit of time depending on the size of the required packages.

    ```bash
    pip install -r requirements.txt
    ```
    {: .nolineno }


### Add a New Web App

1. Go to the <kbd>Web</kbd> tab.
2. Click <kbd>Add a new web app</kbd> and follow the wizard dialog.
3. When prompted to choose a **Python Web Framework** → choose **Manual configuration**.
4. When prompted to select a Python version → select the same version as installed in the virtual environment earlier.

![PythonAnywhere web app creation](50_new-web-app.png){: }

Once the web app is created, scroll down a little until the **Code** and **Virtualenv** sections and apply the following:
- **Source code** – set to `/home/<username>/sites/mydashapp`.
- **Virtualenv** – set to `/home/<username>/.virtualenvs/mydashapp`

![PythonAnywhere Web tab](80_venv-after.png){: }


### Edit the WSGI Config

This is needed for the WSGI config to properly point to your Dash app:
- On the <kbd>Web</kbd> tab, click the WSGI configuration file link. 
- A file editor will open, scroll down to find the `# +++++++++++ FLASK +++++++++++` section.
- We'll now uncomment the relevant lines and modify them to fit our app:

```python
import sys

# Add your project folder to the path
project_home = '/home/<username>/sites/mydashapp'
if project_home not in sys.path:
    sys.path.insert(0, project_home)

# Import the Dash app and expose its Flask server as "application"
from app import app
application = app.server
```
{: .nolineno }

In the end, the section should look like this:

![WSGI file editing](110-wsgi-after.png){: }

That `application = app.server` line is the key for Dash on PythonAnywhere.  
Click <kbd>Save</kbd> to keep your changes.


### Reload Web App and Test

Go back to the <kbd>Web</kbd> tab and click to **Reload** the web app.

![WSGI file editing](120-reload-web-app.png){: }

Once the reload operation is done, open your browser and navigate to your app's webiste, the path should be `https://<username>.pythonanywhere.com`.

The free plan doesn't allow changing the domain name, but if that's required you can always do some basic testing for a week first, see if you're happy with the general performance and usability of the platform, then upgrade to a better plan later on.


### Environment Variables

If you need to set up environment variables, the recommended method is to create `.env` file and load it in the WSGI file using `python-dotenv`:
- Create a `.env` file: In the root directory of your project, create a file named `.env` and add your key-value pairs (e.g., `DATABASE_URL=your_database_url`).
- Install `python-dotenv` via bash console: `pip install python-dotenv`

```python
import os
from dotenv import load_dotenv

project_home = os.path.expanduser('~/sites/mydashapp')
load_dotenv(os.path.join(project_home, '.env'))

DATABASE_URL = os.getenv('DATABASE_URL')
```
{: file='web_app_wsgi.py' .nolineno }

Alternatively, you can set set vars directly in the start of your WSGI file, before importing your app:

```python
import os

os.environ['DATABASE_URL'] = 'your_database_url'
```
{: file='web_app_wsgi.py' .nolineno }

Then, translate the environment variable into a form that can be used in your app:

```python
DATABASE_URL = os.getenv('DATABASE_URL')
```
{: file='settings.py' .nolineno }

Don't forget to **Save** the file and **Reload** web app after changes.

   <!-- markdownlint-capture -->
   <!-- markdownlint-disable -->
   > **Consider security** – never commit sensitive `.env` files (holding secret keys) to version control (e.g., Git). Add `.env` to your `.gitignore` file to avoid that. The same applies to WSGI file in case you decided to store vars directly in it.
   {: .prompt-warning }
   <!-- markdownlint-restore -->


### Steps Summary

1. `server = app.server` in your Python Dash code.
2. Create venv → install reqs → code in `~/sites/mydashapp`. 
3. Web tab → **Add new web app** → **Manual** → set **Virtualenv** + **Source code**. 
4. WSGI → `application = app.server`.
5. **Reload** web app → visit `https://<username>.pythonanywhere.com`. 


### Troubleshooting

- **Logs** – on the <kbd>Web</kbd> tab, open **Error log** and **Server log** to see import errors, tracebacks, and print outputs.

- **Python versions mismatch** – the Python version chosen for the web app must match the one used to create your virtualenv.

- **Outbound internet on free tier** – free accounts can only call specific allowed external sites. If your Dash app fetches data from arbitrary APIs, you'll likely hit errors and will need to upgrade your plan. Here's the [list of PythonAnywhere's allowed sites][pythonanywhere-whitelist].


## Updating PythonAnywhere Web App

By default, PythonAnywhere does not automatically pull updates from GitHub. There are a few ways to do it manually, and it's also possible to automate with a webhook / GitHub-Actions workflow, though I'll leave the webhook approach for another post. It's more complex and requires some fiddling and extra configurations.


### Manual Pull

After updating your GitHub repo, open a bash console in PythonAnywhere and run:

```bash
cd ~/sites/mydashapp
git pull origin main  # change 'main' to your branch name if needed
touch /var/www/synvan_pythonanywhere_com_wsgi.py  # change the wsgi filename to fit yours
```
{: .nolineno }

The `touch` command is equivalent to reloading the web app from the <kbd>Web</kbd> tab manually.  

This approach is simple and straightforward, works every time, though it's manual.
Here's a one-line command to achieve it slightly faster:

```bash
cd ~/sites/mydashapp && git pull origin main && touch /var/www/synvan_pythonanywhere_com_wsgi.py
```
{: .nolineno }


### Manual Pull via Shell Script

Instead of typing this long command each time, we can store it in a shell file and simply run it whenever we need to update our app (less typing and easier to remember just the filename).

1. Create a small shell script so you don't have to type it every time:
    ```bash
    nano ~/update_app.sh
    ```
    {: .nolineno }
2. Paste into your new file:
    ```bash
    #!/bin/bash
    cd ~/sites/mydashapp
    git pull origin main
    touch /var/www/synvan_pythonanywhere_com_wsgi.py
    echo "App updated and reloaded!"
    ```
    {: .nolineno }
3. Save the file: `Ctrl+O` → `Enter` → `Ctrl+X`.
4. Make it executable:
    ```bash
    chmod +x ~/update_app.sh
    ```
    {: .nolineno }

Now, whenever you push to GitHub, just go into PythonAnywhere console (or access it via SSH) and run:
```bash
~/update_app.sh
```
{: .nolineno }


### Pull & Reload on Schedule

PythonAnywhere has a **scheduled tasks** feature:
1. Go to **Tasks** → **Scheduled tasks**.
2. Set the time of the day the task should run.
3. Set the command to run our created shell script `~/update_app.sh`.
4. Give it a description (optional) and create the task.


## Wrap-up

That's it – web app is online. You can have a look at my side project live on PythonAnywhere [here][dps-app]. PythonAnywhere isn't the most powerful or feature-rich option out there, but for lightweight apps it's reliable, simple, and beginner-friendly. If you've got a side project sitting locally on your machine, I'd recommend giving the free tier a try.

[dps-app]: https://synvan.pythonanywhere.com/
[pythonanywhere-whitelist]: https://www.pythonanywhere.com/whitelist/
