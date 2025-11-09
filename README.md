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
