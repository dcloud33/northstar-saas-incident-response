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
`curl -i http://127.0.0.1:5001/health`

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


**Limitations at the Time of Test 2(Historical):**
- Nginx hasn't been configured, so customer-facing access through the reverse proxy hasn't been verified.
- Gunicorn isn't managed by systemd yet.
- Monitoring and service recovery haven't been tested.

**Current Implementaton Status**
- Nginx reverse proxy configured and verified to forward HTTP requests to Gunicorn.
- Gunicorn managed by systemd, with automatic recovery successfully tested
- Python health monitoring implemented and tested.
- State transitions and observed recovery intervals and state changes actively monitored and verified.
- Operational notifications and automated regress tests still pending

## Nginx Reverse Proxy Deployment

## Purpose
Nginx receives HTTP requests and forwards them accordingly
### Configuration Location
```bash
 /etc/nginx/sites-available/northstar
```
### Deployment Procedure
Copy git-tracked configuration into nginx sites-available directory:
```bash
sudo cp infra/nginx/northstar.conf /etc/nginx/sites-available/northstar
```
Create a symbolic link:
```bash
sudo ln -s /etc/nginx/sites-available/northstar /etc/nginx/sites-enabled/northstar
```
Disable the default site:
```bash
sudo unlink /etc/nginx/sites-enabled/default
```
Validate Configuration:
```bash
sudo nginx -t
```
Reload Nginx:
```bash
sudo systemctl reload nginx
```
Verify application through /health:
```bash
curl -i http://127.0.0.1/health
```
### Configuration Validation

Check for any difference from original file copied:
```bas
diff -u /etc/nginx/sites-available/northstar infra/nginx/northstar.conf
```
Validate the configuration:
```bash
sudo nginx -t
```
### Application Verification
Validate Application:
```bash
curl -i http://127.0.0.1/health
```
### Rollback Procedure
Scope: Revert the active Nginx configuration to the default site, disabling the custom reverse-proxy routing to the Northstar application.
Impact: Northstar becomes inaccessible throug its configured Nginx entry point on port 80 because requests are no longer forwarded to Gunicorn.
Nginx continues accepting HTTP requests and may serve its default website, while Gunicorn backend can remain operational on its internal listenting port.

Remove the Northstar symbolic link from sites-enabled:
```bash
sudo unlink /etc/nginx/sites-enabled/northstar
```
Recreating the original default site's symbolic link:
```bash
sudo ln -s /etc/nginx/sites-available/default /etc/nginx/sites-enabled/default
```
Validate the configuration:
```bash
sudo nginx -t
```
Reload Nginx:
```bash
sudo systemctl reload nginx
```
Verify the Results:
```bash
curl -i http://127.0.0.1/health
```
