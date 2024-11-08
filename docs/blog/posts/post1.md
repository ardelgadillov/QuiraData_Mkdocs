---
authors:
    - Andres
tags:
    - Dash
    - Render
    - Webapp
    - Deployment
categories:
    - Python
    - Webapp
title: Deploying a Dash Application on Render A Step-by-Step Guide
description: >- 
   I like to create web applications in Python, especially using the Plotly Dash framework. 
   One of my favorite projects is a Sudoku Solver that’s powered by Mixed-Integer Linear Programming 
   (check it out at [sudoku.quiradata.com](http://sudoku.quiradata.com){:target="_blank"}). 
   I understand the excitement of bringing a project to life on the web. 
   In this guide, I’ll walk you through the process I used to deploy my Sudoku app on Render.
date: 2024-11-07
---

# Deploying a Plotly Dash Webapp on Render: A Step-by-Step Guide

I like to create web applications in Python, especially using the Plotly Dash framework. 
One of my favorite projects is a Sudoku Solver that’s powered by Mixed-Integer Linear Programming 
(check it out at [sudoku.quiradata.com](http://sudoku.quiradata.com){:target="_blank"}). 
I understand the excitement of bringing a project to life on the web. 
In this guide, I’ll walk you through the process I used to deploy my Sudoku app on Render.

## Prerequisites

Make sure you have the following:

- **A Render account**: Sign up at [render.com](https://render.com/){:target="_blank"} 
- **A GitHub account and repository**: This should contain your Python Dash application project

---

## Update Python Code and Create requirements.txt

### Setting Up intro point

1. Locate the section of your python application file where the `app` variable is defined. 
It should look something like this:

    ```python linenums="1"
    app = dash.Dash(__name__)
    ```

2. Add the following line immediately after the `app` definition:

    ```python linenums="2"
    server = app.server
    ```

This exposes your Dash app’s Flask server instance, which is necessary for deployment.

### Creating/Updating `requirements.txt`

1. In the root directory of your repository, create a file named `requirements.txt`. 
You can use [pipreqs](https://pypi.org/project/pipreqs/){:target="_blank"} to generate the requirements file based on imports in project.
For example:

    ```text title="requirements.txt"
    numpy==1.26.4
    pandas==2.2.1
    dash==2.17.1
    Pyomo==6.6.0
    highspy~=1.7.1.dev1
    ```

2. Include **gunicorn** at the end of the requirements.txt file. 
Gunicorn is a Python HTTP server that efficiently handles web requests.

    ```text title="requirements.txt"
    numpy==1.26.4
    pandas==2.2.1
    dash==2.17.1
    Pyomo==6.6.0
    highspy~=1.7.1.dev1
    gunicorn
    ```

---

## Setting Up a New Web Service on Render

1. **Log in to Render** and navigate to your [dashboard](https://dashboard.render.com/){:target="_blank"}.
2. Click **+New / Web Service**.
3. Provide the **URL of your Public GitHub repository** containing the Dash app.
4. Configure the service by:
   - Giving your service a unique name.
   - Language: Python 3
   - Branch: The Git branch to build and deploy. In my case *master*
   - Build Command: Render runs this command to build your app before each deploy
     ```bash
     $ pip install -r requirements.txt
     ```
   - Start Command: Render runs this command to start your app with each deploy
     ```bash
     $ gunicorn app:server
     ```

5. Hit **Deploy Web Service** to finalize the setup.

---

## You're Live!

1. Render will now build and deploy your application. This may take a few minutes.
2. Once deployment is complete, you’ll see a live link to your application.

---

Your Dash app is now live on Render and accessible from anywhere. 


