---
layout: post
title: "Monitor Heroku Apps with New Relic"
subtitle: "Example with Flask and Gunicorn"
background: '/img/newrelic_heroku/nr1_heroku_01.png'
---

# Monitoring Heroku Apps with New Relic
For this example, I have a simple Flask application called `datacrunch-consulting` that is deployed to Heroku via GitHub integration.  Starting with GitHub, you can create a repository.  Then, use GitHub Desktop to clone/push code back to GitHub without using the command line. Linking the repository with Heroku allows the application to deploy every time a push is made to your repository, great for continuous integration / continus deployment.

See GitHub repository here: [https://github.com/pnvnd/flaskapp](https://github.com/pnvnd/flaskapp) or use your own Heroku app.

We'll need to make some adjustments to your Heroku `Procfile`:
```
newrelic-admin run-program gunicorn -b "0.0.0.0:$PORT" -w 3 --chdir datacrunch-consulting webserver:flaskapp
```

We'll go over how to setup the following in New Relic One:
1. Application Performance Monitoring (APM) - for application [golden signals](https://sre.google/sre-book/monitoring-distributed-systems/)
2. Browser, Real User Monitoring (RUM) - for user sessions, geographical region, and JavaScript/AJAX errors
3. Synthetics ping check - for monitoring application uptime
4. Logs - to aggregate logs from different services your application uses (logs in context)
5. Alerts - to send notifictions when certain thresholds are met


## New Relic APM Setup
1. Log into Heroku and verify your account by adding a payment method
1. Go to https://elements.heroku.com/addons/newrelic and select `New Relic APM`
1. Add-on plan: `Wayne - Free`
1. App to provision to `datacrunch-consulting`
1. Click `Submit Order Form`
![Heroku Screenshot 1](/img/newrelic_heroku/nr1_heroku_01.png)
1. Wait for the add-on to install.  Note: it'll create an account for you, even you already have one.  We'll need to configure the environment variables to send APM data to the correct account.
1. In your Heroku dashboard, click on the the app with the New Relic APM add-on
1. Click on Settings > Config Vars > `Reveal Config Vars`
1. You should see at least three (3) configuration variables.  If not, add them manually:
 - `NEW_RELIIC_APP_NAME`
 - `NEW_RELIC_LICENSE_KEY`
 - `NEW_RELIC_LOG`
![Heroku Screenshot 2](/img/newrelic_heroku/nr1_heroku_02.png)
1. Edit the `NEW_RELIIC_APP_NAME` to give your application a name. This is what shows up in New Relic.  
You also can edit this in New Relic One > APM > Settings > Application > Application Settings > Application alias
1. Edit `NEW_RELIC_LICENSE_KEY` to have the license key for the New Relic account you want data ingested into
1. Restart your application by toggling off/on your dynos in Heroku > Resources
1. Go to your deployed Heroku app and log into New Relic One and check APM for data ingested.
![Heroku Screenshot 3](/img/newrelic_heroku/nr1_heroku_03.png)

## New Relic Browser Setup
With the APM agent installed on Heroku, you can get Real User Monitoring to track browser sessions, which browser is being used, which landing page the user is getting errors (if any), and geographical location of your users.

1. In New Relic One, click `Add more data` on the top-right and select ` Browser Metrics`
1. Select `Enable via New Relic APM` and select your application from the selection box.
![Heroku Screenshot 4](/img/newrelic_heroku/nr1_heroku_04.png)
1. Once enabled, go to your Heroku app to get data ingested.  Note: Some adblockers prevent JavaScript from running in the background.
![Heroku Screenshot 5](/img/newrelic_heroku/nr1_heroku_05.png)

## New Relic Synthetics Setup
Heroku apps on the free tier shuts down the app after 30 minutes of inactivity.  When someone accesses the Heroku app after inactivity, it will `cold start` and may take a minute to load.  Having a simple synthetics ping test every 15 minutes will keep the Heroku app up to avoid "cold starts".  

Personal Heroku accounts have 550 free dyno hours each month between 5 apps.  If you verify your account with a payment method, you get an additional 450 hours free per month.  That is, 1000 dyno hours over 100 apps.  Using synthetic ping tests can keep 2 Heroku apps running without them ever going to sleep!

1. Go to New Relic One > Synthetics
1. Add monitor by giving it a name
1. For `Text validation (optional)`, add a string `Welcome!` (or whatever string that appears on your endpoint)
1. Advanced options: enable `Verify SSL` to check for expired certificates
1. Check 1 location, every 15 minutes
![Heroku Screenshot 6](/img/newrelic_heroku/nr1_heroku_06.png)
1. Wait for your Synthetics ping check to get back some data to see uptime info
![Heroku Screenshot 7](/img/newrelic_heroku/nr1_heroku_07.png)


## New Relic Logs Setup (Command Line)
General Instructions here: https://docs.newrelic.com/docs/logs/forward-logs/heroku-log-forwarding/

1. Download and install the Heorku CLI:  
`https://devcenter.heroku.com/articles/heroku-cli`

2. Check if Heroku is installed:  
`heroku --version`

3. Update Heroku CLI as needed:  
`heroku update`

4. Log in:  
`heroku login`
![Heroku Screenshot 13](/img/newrelic_heroku/nr1_heroku_13.png)

5. Check Environment variables, in case you can't get to Heroku Dashboard > Settings:  
PowerShell: `heroku run env --app datacrunch-consulting | findstr -i NEW_RELIC`
![Heroku Screenshot 12](/img/newrelic_heroku/nr1_heroku_12.png)

6. If you need to set environment variables from the command line:
```
heroku config:set NEW_RELIC_LOG='stdout' --app datacrunch-consulting
heroku config:set NEW_RELIC_APP_NAME='flaskapp.heroku' --app datacrunch-consulting
heroku config:set NEW_RELIC_LICENSE_KEY='a1b2c3d4e5f6g7h8i9j0NRAL' --app datacrunch-consulting
```

7. Add Drain Logs  
`heroku drains:add syslog+tls://newrelic.syslog.nr-data.net:6515 --app datacrunch-consulting`

8. Copy drain token:  
`heroku drains --app datacrunch-consulting --json`
![Heroku Screenshot 11](/img/newrelic_heroku/nr1_heroku_11.png)

9. Add drain token in New Relic
![Heroku Screenshot 8](/img/newrelic_heroku/nr1_heroku_08.png)

10. Add tokens and restart Heroku Dynos: Heroku Dashboard > More > Restart all Dynos or:  
`heroku dyno:restart --app datacrunch-consulting`

11. Go to New Relic One > Logs > Patterns to see logs, and any aggregated log messages based on patterns:
![Heroku Screenshot 9](/img/newrelic_heroku/nr1_heroku_09.png)

12. You can also `Query your data` with NRQL to set up alerts based on log messages
![Heroku Screenshot 10](/img/newrelic_heroku/nr1_heroku_10.png)

## New Relic Alerts Setup
1. In New Relic One, click on `Alerts & AI` > `Notification Channels`
2. On the top-right, click `New notification channel` and add a notification channel of your choice
![Heroku Screenshot 14](/img/newrelic_heroku/nr1_heroku_14.png)
3. Next, go to `Alerts & AI` > `Policies` > `New alert policy`
4. Git your Alert Policy a Name `Create alert policy`
![Heroku Screenshot 15](/img/newrelic_heroku/nr1_heroku_15.png)
5. Click on `Create a condition`
6. Alerts for APM, Browser, and Synthetics are straight-forward and have selections to choose from.  We'll work with NRQL alerts then click `Next, define thresholds`
7. The NRQL query we'll use for alerting will be based on any log messages from Heroku that contain "Shutting down" at least once in 5 minutes.  This will send an alert when the app shuts down on Heroku for whatever reason.
```
SELECT count(*) from Log where plugin.type='syslog-heroku' and message like '%Shutting down%'
```
![Heroku Screenshot 16](/img/newrelic_heroku/nr1_heroku_16.png)