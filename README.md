# Northstar SaaS Incident Response

## Buisness Problem
The customers cannot consistently access a service they depend on.

## Buisness Impact
The customers may lose faith in the company, due to the fact they can't access their resources
and seek service else where.


## Project Objectives
Nginx should recieve the customer traffic and not the Gunicorn(backend)
Create an application failure and verify the monitoring system detects it and records the event
Nginx recieves HTTP requests on port 80
Gunicorn runs our Flask application on port 5001



## Success Criteria
Successful HTTP response through Nginx
Monitoring to detect a simulated failure
Simulate an application failure, verify the service becomes unavailable, restore the application confirm
an HTTP 200 response with the expected content through Nginx, and record the elapsed recovery time
Backend is not directly accessible from external networks

