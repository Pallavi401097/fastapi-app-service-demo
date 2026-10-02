# Deploy a Python (FastAPI) web app to Azure App Service (Free F1 plan)

A minimal FastAPI app (with a Jinja2 template and a form) that you can deploy to
**Azure App Service on Linux** using the **Free F1** pricing tier, through the Azure Portal
and Visual Studio Code.

This version of `main.py` is updated for current FastAPI/Starlette releases: it calls
`templates.TemplateResponse(request, "index.html")` (request first). The older call style
`TemplateResponse("index.html", {"request": request})` causes
`TypeError: unhashable type: 'dict'` and an "Internal Server Error" on Azure.

## Project structure

```
.
├── main.py             # FastAPI app (routes: /, /hello, /favicon.ico)
├── requirements.txt    # Python dependencies
├── startup.sh          # Startup command used on Azure
├── static/             # Bootstrap, favicon, images
└── templates/          # index.html, hello.html
```

`main.py`, `requirements.txt`, `static/` and `templates/` must be at the **root** of the
folder you deploy (not inside another sub-folder).

## Prerequisites

- An Azure account (a free account works): https://azure.microsoft.com/free/
- Python 3.11 or newer
- [Visual Studio Code](https://code.visualstudio.com/) with the
  **Azure App Service** extension (and the Python extension)
- Git and a GitHub account (only if you want to push the code to GitHub)

## 1. Test locally (optional)

```bash
pip install -r requirements.txt
uvicorn main:app --reload
```

Open http://127.0.0.1:8000/ in your browser.

## 2. Create the App Service (Azure Portal)

1. Sign in to https://portal.azure.com.
2. Search for **App Services** in the top search bar and click **Create > Web App**.
3. On the **Basics** tab:
   - **Subscription**: choose yours.
   - **Resource Group**: click **Create new** and give it a name (for example `fastapi-rg`).
   - **Name**: enter a globally unique app name (this becomes `https://<name>.azurewebsites.net`).
   - **Publish**: **Code**.
   - **Runtime stack**: **Python 3.11** (or newer).
   - **Operating System**: **Linux**.
   - **Region**: choose a region near you. If the Free F1 plan is not available, try another region.
4. Under **Pricing plans**, create a new Linux plan and select **Free F1**
   (it may appear under the **Dev/Test** tab of the pricing plan picker).
5. Click **Review + create**, then **Create**.
6. When the deployment finishes, click **Go to resource**.

## 3. Set the startup command

Azure does not know how to start a FastAPI app by default, so this step is required.

1. In your web app, go to **Settings > Configuration** (look under **Stack settings** or
   **General settings**, depending on the portal version).
2. In **Startup Command**, enter:

   ```
   python -m uvicorn main:app --host 0.0.0.0
   ```

   (This is the same command that is in `startup.sh`.)
3. Click **Save** and let the app restart.

## 4. Deploy the code from Visual Studio Code

1. Open the project folder in VS Code (the folder that contains `main.py`).
2. Install the **Azure App Service** extension if you have not already, and sign in to
   Azure from the Azure icon in the left sidebar.
3. Expand your subscription, find your web app, then **right-click it > Deploy to Web App**.
4. Select the project folder when asked, and confirm if it asks to overwrite the existing deployment.
5. Wait for the message **Deployment successful**.

### Alternative: deploy from GitHub

1. Create a GitHub repository (for example `fastapi-azure-webapp`) and push this project so
   that `main.py` is at the repo root.
2. In the Azure Portal, open your web app > **Deployment > Deployment Center**.
3. Choose **GitHub** as the source, authorize, then select your organization, repository
   and branch (for example `main`), and click **Save**.
4. Azure adds a GitHub Actions workflow that builds and deploys the app on every push.

## 5. Restart and browse the app

1. In the Azure Portal, open your web app and click **Restart** on the **Overview** page.
   Wait about a minute.
2. Click the **Default domain** link on the Overview page
   (`https://<your-app-name>.azurewebsites.net`).
3. You should see the **Welcome to Azure** page. Enter a name and click **Say Hello** to test
   the `/hello` route.
4. You can also open `https://<your-app-name>.azurewebsites.net/docs` for FastAPI's
   interactive Swagger UI.

## Troubleshooting

- **Internal Server Error**: open **Monitoring > Log stream** in the portal, scroll to the
  very bottom, load the site, and read the newest `Traceback`. The log stream replays old
  entries first, so only trust lines that appear after your latest deployment.
- **Check that the new code is deployed**: go to **Deployment > Deployment Center > Logs**
  and confirm the latest deployment is marked **Succeeded (Active)**. The portal shows your
  local time and the logs show UTC.
- **`TypeError: unhashable type: 'dict'`**: you are running the old `main.py`.
  Use `templates.TemplateResponse(request, "index.html")` and redeploy.
- **Default Azure page or app does not start**: make sure the **Startup Command** from
  step 3 is set and saved.
- **Site not updating after a redeploy**: click **Restart** on the Overview page and do a
  hard refresh in the browser (Ctrl+Shift+R).

## Free F1 plan limits

- 60 CPU minutes per day, 1 GB RAM and 1 GB storage (shared compute).
- No SLA, no custom domain, no deployment slots.
- The app can sleep when idle, so the first request after a while may be slow.
- If you exceed the daily CPU quota, the app is stopped until the quota resets.
- Intended for testing and learning. For production, use a paid plan such as B1.

## Clean up

To avoid leftover resources, delete the resource group (**Resource groups > your group >
Delete resource group**) when you no longer need the app.

## Next steps

- FastAPI documentation: https://fastapi.tiangolo.com/
- Azure App Service Python quickstart: https://learn.microsoft.com/azure/app-service/quickstart-python
