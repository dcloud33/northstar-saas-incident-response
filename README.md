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

## Application Verification

### Test 1: Flask Health Endpoint

**Objective:** Verify that the Flask application responds
successfully to an HTTP health-check request.

**Command:**
`curl -i http://1127.0.0.1:5001/health`

**Expected Results:**
- HTTP status code: 200
- Content-Type: application/json
- Response body: {"status": "healthy"}

**Actual Results:**
- HTTP status code: 200
- Content-Type: application/json
- Response body: {"status":"healthy"}

**Test Status:** PASSED


### Test 2: Gunicorn Application Server

**Objective:** Verify that the Gunicorn application
responds to an HTTP health-check request and run 2 Gunicorn workers.

**Configuration:**
`gunicorn --workers 2 --bind 127.0.0.1:5001 app:app > /tmp/northstar-gunicorn.log 2>&1 < /dev/null &`

**Verification commands:**
`curl -i http://127.0.0.1:5001/health`
`ps -o pid,ppid,stat,cmd -p 3185,3190,3191`

**Actual results:**
- HTTP status code: 200
- Server: gunicorn
- Content-Type: application/json
- Response body: {"status": "healthy"}

**Test status:** PASSED


**Limitations:**
- Nginx hasn't been configured, so customer-facing access through the reverse proxy hasn't been verified.
- Gunicorn isn't managed by systemd yet.
- Monitoring and service recovery haven't been tested.
