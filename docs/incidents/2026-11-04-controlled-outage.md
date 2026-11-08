## Incident Summary

Purpose: This was a controlled application failure test

Failure: Gunicorn was stopped and resulted in application-health failure

Detection: Python monitoring system checks resulted in an UNHEALTHY status

Recovery: I restarted the Gunicorn service, checking the logs, ran HTTP request to the application to verify that it was available

Outcome: the observed 34.49-second detection-to-recovery interval, doesn't represent an exact downtime

## Customer Impact
During the controlled outage, the application's /health endpoint stopped returning successful health-check responses because
the Gunicorn backend was unavailable. In a live production environment, this exact type of backend failure could disrupt crititcal business workflows,
preventing customers from logging in or loading their account dashboards. Because these observations were captured during a planned experiment
inside a local Ubuntu lab environment, they represent a simulated technical failure rather than a verified production incident.

## Timeline

| Time (UTC) | Event | Evidence Source |
|---|---|---|
| 22:46:15.995 | Last successful health check before the experiment | Python Monitor |
| 22:46:16 | Shutdown Initiated | systemd journalctl |
| 22:46:17.006 | Python monitoring detected the first unhealthy response | Python Monitor |
| 22:46:50 | Service started | systemd journalctl |
| 22:46:51.495 | First healthy response following restoration | Python Monitor |
## Root Cause
The Northstar service interruption was initentional administrative acton when a system administrator manually stopped the Gunicorn process. Because Nginx 
acts as a reverse proxy that relies on Gunicorn to handle dynamic application logic, stopping the backend left Ngix with nowhere to forward incoming
requests, causing it to return errors to the client.

## Recovery
- Gunicorn service was manually restarted using
```bash
sudo systemctl start northstar.service
```
- Recovery was verified through
```bash
systemctl is-active northstar.service
```
- A successful HTTP 200 response containing the expected health-check JSON, with the command
```bash
curl -i http://127.0.0.1/health
```


## Lesson Learned
What Worked: The automated python monitoring script successfully detected the outage in real time and and precisely
logged the transition from healthy to unhealthy states.

Limitation Discovered: Relying solely on a generic /health endpoint createds an evidence boundary blind spot, meaning we cannot definitively confirm
the failure status of specific, untested features like login or dashboard functionality.

Operational Improvement Recommended: Implement distinct synthetic monitoring checks for critical user paths and configure a robust
operational alerting system to notify engineering teams immediately during a backend failure.
