# IOT Data Stale

**Date**: 01/30/2020

**Status**: ⚠️ Mitigated

After a major deployment of IOT data software that completely changed the infrastructure it runs on, the company's Data Analyst verbally warned that data was not flowing properly. This was caused by several issues with the new deployment that range from incompatibilities in technology, to faulty configurations.

## Overview

A [deployment of IOT etl](https://teams.microsoft.com/) was scheduled for January 30th, after completion of [an issue](https://github.com/) opened after a [previous incident](https://github.com/). The issue being resolved had to do with consolidating the executable that runs the IOT data etl process. As a result, deployment of this newer version of the IOT data etl process completely changed the operational procedure and architecture of the application's infrastructure. This required a [completely new deployment](https://github.com/) and the removal of previous deployments.

## Impact

The IOT data etl process was heavily impacted by this outage. This process is responsible for recording and aggregating telemetry data for equipment out in the field. The output of this process provides workers in the field with a comprehensive view of inventory in the field and assists them in making critical business and operational decisions. The severity of the impact and revenue loss that resulted from this particular outage is unknown at this time.

## Root Cause

The root cause of this incident is down to a faulty deployment of a new architecture of the application. This comes down to two major issues: Vendor A HTTP308 issues and Vendor B HTTP401 issues.

Vendor A - Supplies IOT devices and broadcasts telemetry for our service to capture
Bendor B - Supplies Full equpiment solution and provides an api to retrieve telemetry data.
Vendor C - Supports company's proprietary flagship application

### Vendor A HTTP308 issues

The new webserver that the IOT data etl process api was deployed to requires that clients (Vendor A) use HTTPS. We had previously had Vendor A configured to use plain HTTP. The new web server was configured to forward clients who use HTTP to HTTPS via a permanent redirect. When you're dealing with individual users (i.e. humans), this permanent redirect is seamless and the browser abstracts all complexity away for the user. However, when you're dealing with machine to machine communication, permanent redirects (HTTP Status code 308) are not always followed and a process might stop and error out to prevent any session hijacking or other malicious activities taking place.

### Vendor B HTTP401 issues

The issues that were presented with the Vendor B data was not immediately apparent. In fact there was no issue with the Vendor B integration until after an hour of runtime. After one hour of runtime, Vendor B data stopped being updated and the application was raising a `401 (Unauthorized)` HTTP error, which is an error raised when login credentials are incorrect or a user's session expires.

## Detection

Immediately after deployment of the new IOT data etl process version which took place in the early afternoon of Thursday, January 30th, The company's Data Analyst let the software engineers know that data was going stale. The software engineers were not too concerned as the deployment included a CNAME record change and those things can take several hours to resolve. Company software Engineers reached out to Vendor A (the affected vendor) and requested that they initiate reboots to the IOT devices in the field in hopes this would invalidate DNS cache and resolve them to the new web server. Company Software Engineers did not fully see that effort through due to leaving the office early for various personal reasons. At this time, it had not yet manifested that Vendor B was also affected, as their failure mode was delayed by one hour of run time due to the nature of the issue affected the secondary vendor.

The next morning, the company's Data Analyst again raised the alarm [in microsoft teams](https://teams.microsoft.com/), letting the team know that no data was being received, this time stressing the fact that all vendors were affected.

An additional problem occurred where Company's Proprietary Flagship Software did not properly indicate that IOT was stale. Vendor C made a design decision that was directly against what was asked for from a design perspective where the only indication that IOT data was stale was a slight change in the border of the bars on the bar graph to be bolded grey when data was stale instead of black when data was up-to-date. The original ask was to make it abundantly clear that data was stale and this bolded grey border was easy to miss.

## Resolution and Recovery

### Vendor A HTTP308 resolution

We requested Vendor A either follow the HTTP 308 redirect, which is undesirable and a security risk, or switch over to using HTTPS instead. Vendor A started switching over to HTTPS, but started to encounter a `sslv3_alert_handshake_failure`. Vendor A is still investigating this problem, but as a short-term solution, we have started the old web server back up and have Vendor A sending data there instead. This will buy Vendor A a few days of time to resolve their SSL handshake problems.

### Vendor B HTTP401 resolution

Initially it was presumed that the app did not have the correct password to make api requests to the Vendor B API. This was corroborated upon investigation when it was found that the [sealed secret](https://github.com/bitnami-labs/sealed-secrets) was configured with the incorrect Environment Variable Key `VendorB_password`. The correct Environment Variable Key would have been `VendorB_API_Password`. The error with the environment variable was corrected, however it would not have an impact on the issue at hand. When the _exact_ environment variable key for a certain value cannot be found, the application will use it's default. Since [this issue](https://github.com/), which is to remove production credentials from production, has not yet been completed, the application will default to live, valid, production credentials.

The actual issue was determined that the login session to the Vendor B API was expiring after one hour. The process only logs into the Vendor B API once at startup and reuses the same session for the lifetime of the application. Since sessions are only valid after one hour, 401 errors would be experienced after one hour of run time. The previous version of the process did not fall victim to this fault because a new instance of the application was created with each payload retrieved from the Vendor B API, and as a result, sessions never lasted more than a few minutes. This was resolved in [this github issue](https://github.com/) and resulting [pull requests](https://github.com/).

The reason the password was originally though the be faulty even after an hour of successful runtime was due to The Company's technical team being unfamiliar with kubernetes sealed secrets.

### Action Items

Status | Action Item | Type | Owner | Link
---: | --- | :---: | ---: | ---
🔄 | Improve Data Warehouse alerts for stale IOT data | Strategic | Data Analyst | [Github Issue](https://github.com/)
🔄 | raising the flag when a IOT device gets turned off. | Tactical | Data Analyst |
🔄 | Update software to support HTTPS | Strategic | Vendor A | [Github Issue](https://github.com/)
🔄 | Create/Enable testing tools | Strategic | Vendor A | [Github Issue](https://github.com/)
🔄 | Create/Enable testing environment | Strategic | DevOps | [Github Issue](https://github.com/)
🔄 | Incident Management Tool / Known issues / Status Page / Alerts | Strategic | Software Engineering Team |
🔄 | Implement Go/NoGo procedures | Strategic | Software Engineering Team |
🔄 | Require rollback plan, and parameters that require rollback | Strategic | Software Engineering Team |
🔄 | Address people relying on HMI pictures and not IOT data | Tactical | Software Engineering Team |
🔄 | Communicate with Vendors when updates take place | Tactical | Software Engineering Team |

## Lessons Learned

### What went well

- Quick response times
- Data Analyst raised the alarm and noticed it on the day they were off.
- Quick deployments after solutions were found.

### What went wrong

- 18 hours before a serious response was mounted
- Misdiagnosis of initial report.
- Communication could be improved
  - Team members need to acknowledge NOTAMs
  - Acknowledging members need to present a plan and try to provide a rough timeline
  - Executing team members need to provide ongoing feedback into progress.
- When an action item is taking place, a timeline should be provided
- More Testing for deployments
- More communications with vendors.
- Data Warehouse stale IOT data alerts are heavy handed, not as refined as they could be. A lot of false positive alerts.

### Where we got lucky

- We did not cause NPT (non-production time). Probably because people are still not using IOT data in legacy proprietary flagship application, they are still relying on HMI pictures.
- Keeping the AWS Elastic Beanstalk instance up and available allowed CNAME record rollback

## Timeline

> All times in CST

Time | Description
--- | ---
1/28 | 
12:45 | Software Engineer 1 submitted a [pull request](https://github.com/) to resolve a major issue.
15:33 | Software Engineer 2 approved and merged the pull request
1/30 | 
11:00 | Software Engineer 1 & 2 have a discussion with a Revelry DevOps Engineer to deploy the previously merged feature
11:41 | Software Engineer 2 submits a [pull request](https://github.com/) to deploy the new features.
12:00 | Software Engineer 2 discovers that the old container was deployed instead of the new one. Effectively changing nothing in the architecture of the system. It was decided to break for lunch and try again in the afternoon.
14:03 | Software Engineer 2 submitted a new deployment [pull request](https://github.com/)
14:04 | Software Engineer 1 approves the new deployment pull request.
14:21 | Software Engineer 2 merges the pull request which causes the infrastructure changes to take place
14:21 | Software Engineer 2 makes the CNAME record change in GoDaddy to point Vendor A to the new correct api endpoint for IOT data etl process
14:25 | Software Engineer 2 submitted a [pull request](https://github.com/) to prepare for deployment.
14:33 | Software Engineer 1 & 3 approved and merged the pull request.
15:00 | Software Engineer 1 leaves for the day.
15:15 | ⭐ Company Data Analyst informs Software Engineer 2 that data has gone stale.
15:20 | Software Engineer 2 Alerts Vendor A CIO that no data is being received
15:27 | Software Engineer 2 and Vendor A agree a CNAME record change could cause issues with Caching, a system reboot of the field unit is initiated and it is assumed this will be sufficient to resolve the DNS Caching issue.
15:30 | Software Engineer 2 leaves for the day.
16:06 | Vendor A CIO responds to Software Engineer 2 informing that the reboots have been completed, and request that further monitoring be done.
1/31 | 
08:04 | ⭐ Company Data Analyst reports via Microsoft Teams that IOT data has been stale for over 17 hours.
09:00 | Software Engineer 2 arrives. Company CIO Immediately informs that IOT data etl process is down.
09:18 | Software Engineer 2 informs Vendor A CIO that data is still not available
09:27 | Software Engineer 2 opens discussion with Vendor C Engineers about potential issues with IOT data etl process.
09:31 | Vendor C confirms that the webserver is up and accessible
09:37 | Communication over Vendor B Environment variables being and it is discovered that a key value is wrong.
09:48 | Invalid environment variable key name discovered.
09:54 | Correction to the Vendor B Environment variable name is made into a [pull request](https://github.com/)
09:55 | Vendor A begins to run traces on the protocol pipeline
10:02 | Vendor A CIO calls Software Engineer 2 to discuss traces
10:04 | Software Engineer 2 requests Vendor C DevOps engineer joins the phone call
10:05 | Vendor C DevOps engineer joins the call with Vendor A CIO and Software Engineer 2
10:10 | It is discovered that Vendor A is using HTTP instead of HTTPS. Vendor A begins making relevant changes.
10:13 | Correction to Vendor B Environment variable name is [merged](https://github.com/). This restarts the pods. the 401 Unauthorized error disappears
10:35 | Software Engineer 2 misses call from Vendor A CIO. Vendor A CIO send text message to Software Engineer 2 requesting call back.
10:44 | Software Engineer 2 calls Vendor A CIO. Compatibilities with HTTPS have been discovered and present a major hurdle.
10:58 | Vendor A CIO calls Software Engineer 2 back after dropped call.
11:00 | A meeting among all concerned tech contacts with the company is called to discuss the ongoing incident and preventing future incidents.
11:14 | Vendor B Integrations starts failing with 401 Unauthorized again. Undetected
11:35 | Software Tech meeting is adjourned.
11:37 | Software Engineer 2 calls Vendor A CIO to confirm next steps, future prevention, and further discuss the ongoing issue.
12:00 | Software Engineer 2 breaks for lunch, awaiting feedback from Vendor A on the ongoing issue.
01:16 | Software Engineer 2 informs Vendor C DevOps Engineer that the Vendor B issue is back and a restart of the pods is requested.
01:47 | Vendor C DevOps engineer restarts the pods with an apology for the delay in response.
01:56 | Software Engineer 2 confirms the restart resolved the issue with Vendor B data. Requests instructions to do the pod restart themselves.
14:03 | More discussion about the faulty Environment variable name, assumptions that it reverted back to the faulty state somehow. Mostly confusion.
14:05 | Software Engineer 2 Responds to email received by Vendor A Engineer
14:10 | Vendor A CIO calls Software Engineer 2 to discuss challenges with switching over to HTTPS. Agreement to remain on HTTP in the old environment temporarily
14:15 | Software Engineer 2 updates the CNAME record to point back to the old environment that could accept HTTP.
14:26 | ⭐ Data from Vendor A is confirmed to be flowing. The issue with Vendor A has been Mitigated
14:30 | Vendor C DevOps engineer offers additional help setting up a less secure Ingress for the new environment. Software Engineer 2 declines the offer, choosing to not introduce additional unsecured standards to the environment.
14:45 | Vendor C DevOps engineer provides instructions on how to perform a pod/namespace restart in the company Cluster.
14:47 | Vendor B Integrations starts failing with 401 Unauthorized again. Undetected
16:19 | Software Engineer 1 notices stale Vendor B data again.
16:20 | Software Engineer 2 Investigates Vendor B pods with Vendor C DevOps engineer, still suspecting Environment variables are to blame
16:30 | Software Engineer 2 realizes the consistent 1 hour run times and realizes its a session expiration issue. Software Engineer 2 Discusses getting a hotfix in place with Software Engineer 1.
16:31 | Vendor C DevOps Engineer restarts the pods to get another hour session with the Vendor B API.
17:25 | Software Engineer 2 publishes a [pull request](https://github.com/) to refresh the Vendor B sessions when the expire.
17:31 | Vendor B Integrations starts failing with 401 Unauthorized again. The recent pull request has not yet been deployed.
17:34 | The hotfix is deployed. Software Engineers commute home and plan to monitor it remotely.
18:35 | Vendor B Integrations starts failing with 401 Unauthorized again, even after the first hotfix.
19:48 | It's discovered a small mathematical error was made in calculating time offsets. A hotfix is prepared and a [pull request](https://github.com/) is submitted.
19:51 | The recent hotfix is deployed and the pods are restarted.
22:08 | ⭐ After over two hours of monitoring the Vendor B data. the 401 Unauthorized error has not returned.

## Supporting information

- links to slack chats
- links to log files
- links to research material used for resolution
- etc.
