# Deploying .NET MVC / Web API with PostgreSQL on Render
This document provides two methods for deploying a .NET MVC or Web API project with a PostgreSQL database on Render:


1. Classic/manual setup
2. Automated render.yaml approach


## 1️⃣ Classic / Manual Setup

### Step 1: Prepare Your Project

Run locally to ensure it works:

```bash```
dotnet run

Ensure .NET 6/7/8 is targeted in your .csproj:

<TargetFramework>net8.0</TargetFramework>

Set a placeholder database connection string in appsettings.json:

"ConnectionStrings": {
  "DefaultConnection": "Host=localhost;Port=5432;Database=MyAppDB;Username=postgres;Password=yourpassword"
}

Step 2: Push to GitHub

git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/username/repo.git
git push -u origin main

Step 3: Sign Up / Login to Render

1. Go to: https://render.com


2. Login via GitHub


3. Authorize access to your repository



Step 4: Create Web Service

Dashboard → New + → Web Service → Select GitHub repo

Settings:

Name: my-dotnet-app

Runtime: dotnet

Branch: main


Build Command:

dotnet publish -c Release -o ./publish

Start Command:

dotnet ./publish/YourApp.dll

Step 5: Create Free PostgreSQL Database

Dashboard → New + → PostgreSQL → Name: myapp-db → Free plan

Copy the database URL.

Step 6: Add Environment Variable

Dashboard → Web Service → Environment → Add

Key: ConnectionStrings__DefaultConnection

Value: <DATABASE_URL_FROM_RENDER>


Step 7: Apply EF Core Migrations

Locally:

dotnet ef migrations add InitialCreate
dotnet ef database update

Or in Render shell:

dotnet ef database update

Step 8: Test Your App

Open URL: https://my-dotnet-app.onrender.com




## 2️⃣ Automated render.yaml Approach

Step 1: Create render.yaml

services:
  - type: web
    name: my-dotnet-app
    env: dotnet
    branch: main
    plan: free
    buildCommand: dotnet publish -c Release -o ./publish
    startCommand: dotnet ./publish/YourApp.dll
    autoDeploy: true
    envVars:
      - key: ConnectionStrings__DefaultConnection
        fromDatabase:
          name: myapp-db
          property: DATABASE_URL

databases:
  - name: myapp-db
    databaseName: myappdb
    user: postgres
    password: strongpassword123
    plan: free
    region: oregon

Step 2: Push render.yaml to GitHub

git add render.yaml
git commit -m "Add Render IaC config"
git push origin main

Step 3: Deploy from Render YAML

Dashboard → New + → Connect via render.yaml → Select repository

Render reads render.yaml and provisions:

Web Service

PostgreSQL database

Environment variable


Step 4: Test App

Open URL: https://my-dotnet-app.onrender.com







# ⚙️ Windows-based GitHub Actions Workflow for Render Deployment

## Step 1: Create file in proj root => .github/workflows/deploy.yml

## 🧾 Step 2: Add the following workflow content
⬇️

name: Deploy .NET App to Render (Windows)

on:
  push:
    branches:
      - main

jobs:
  build-and-deploy:
    runs-on: windows-latest

    steps:
    # 1️⃣ Checkout code
    - name: Checkout repository
      uses: actions/checkout@v4

    # 2️⃣ Setup .NET SDK
    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: '8.0.x'

    # 3️⃣ Restore dependencies
    - name: Restore dependencies
      run: dotnet restore

    # 4️⃣ Build project
    - name: Build
      run: dotnet build --configuration Release --no-restore

    # 5️⃣ Run tests (optional)
    - name: Run tests
      run: dotnet test --no-build --verbosity normal

    # 6️⃣ Publish build output
    - name: Publish
      run: dotnet publish -c Release -o ./publish

    # 7️⃣ Trigger Render Deployment
    - name: Trigger Render Deployment
      env:
        RENDER_SERVICE_ID: ${{ secrets.RENDER_SERVICE_ID }}
        RENDER_API_KEY: ${{ secrets.RENDER_API_KEY }}
      run: |
        curl -X POST "https://api.render.com/v1/services/$env:RENDER_SERVICE_ID/deploys" `
        -H "Authorization: Bearer $env:RENDER_API_KEY" `
        -H "Accept: application/json" `
        -H "Content-Type: application/json"


  ##  🔑 Step 3: Set Up GitHub Secrets

    Go to your repository → Settings → Secrets and variables → Actions → New repository secret

Add these:

Secret Name	Description

RENDER_API_KEY	From your Render dashboard → Account Settings → API Keys
RENDER_SERVICE_ID	From Render → Open your service → Copy from service URL (or via API list)



## ⚙️ Step 4: How It Works

Every push to main runs the workflow.

The .NET project is built and tested on GitHub servers.

After successful build, a new deployment is triggered on Render using the Render API.

Render will pull from GitHub and deploy automatically (same as a manual trigger)
