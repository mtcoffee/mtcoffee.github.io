---
title: ServiceNow ITOM and n8n
---

# Summary
If you haven’t taken [n8n](https://n8n.io/) for a test run yet, you should give it a shot. It’s free, easy to set up in Docker, and quick to get running.

**n8n** is essentially a workflow automation tool that takes input, processes it, and produces output, all on a clean, visual canvas. The possibilities are endless. You can use it for IT operations, security automation, integrations, or even simple monitoring tasks. For example, I’ve recently started using it to monitor stock tickers and notify me when my preferred strike prices are reached.
# Sample ServiceNow Event Use Case
You can use **n8n** to build a custom monitoring solution. In the example below, I store a list of endpoints in a table within the n8n instance. When the workflow runs, it checks each endpoint, and if it doesn’t return a `200` response code, the workflow marks the host as down in the table and sends a request to the ServiceNow Event API;  `https://instance.service-now.com/api/global/em/jsonv2`, with a JSON payload to create an event. When the endpoint comes back online, a clear event is sent to ServiceNow.

This type of monitoring can be extended to almost anything since n8n workflows support programmatic actions like SSH, Python scripts, or web requests.


# Sample n8n Workflow and Resulting Alert
I have brought down my home-assistant container to demonstrate.

![]({{ 'assets/images/n8n.png' | relative_url }})

![]({{ 'assets/images/serviceNown8nevent.png' | relative_url }})
