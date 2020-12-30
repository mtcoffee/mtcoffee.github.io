---
title: Presentable ServiceNow Notifications
---

Out of the box, the ServiceNow notifications are not the most attractive. ServiceNow's own notifications do not use anything that resembles the default layout. The most common method is to use html tables to ensure that email layouts are consisent across devices and clients (looking at you Outlook!). Let's create a better look.

# Create a new Layout
1. Under System Policy -> Email -> Layouts, click New.
2. Title it "**Sample Layout**"
3. Click the html code button "<>" in the layout bar and paste in the html below.

```
<style type="text/css">
        /* CLIENT-SPECIFIC STYLES */
        body, table, td, a { -webkit-text-size-adjust: 100%; -ms-text-size-adjust: 100%; }
        table, td { mso-table-lspace: 0pt; mso-table-rspace: 0pt; }
        img { -ms-interpolation-mode: bicubic; }

        /* RESET STYLES */
        img { border: 0; height: auto; line-height: 100%; outline: none; text-decoration: none; }
        table { border-collapse: collapse !important; }
        body { height: 100% !important; margin: 0 !important; padding: 0 !important; width: 100% !important; }

        /* iOS BLUE LINKS */
        a[x-apple-data-detectors] {
            color: inherit !important;
            text-decoration: none !important;
            font-size: inherit !important;
            font-family: inherit !important;
            font-weight: inherit !important;
            line-height: inherit !important;
        }

        /* MEDIA QUERIES */
        @media screen and (max-width: 480px) {
            .mobile-hide {
                display: none !important;
            }
            .mobile-center {
                text-align: center !important;
            }
        }

        /* ANDROID CENTER FIX */
        div[style*="margin: 16px 0;"] { margin: 0 !important; }

        p {font-family: Open Sans, Helvetica Neue, Helvetica, Arial, sans-serif; font-size: 16px; font-weight: 400; line-height: 24px; color: #444444;}
    </style>
<p><br /></p>
<table border="0" width="100%" cellspacing="0" cellpadding="0">
<tbody>
<tr>
<td style="background-color: #eeeeee;" align="center" bgcolor="#eeeeee">
<table style="max-width: 600px; height: 367px; width: 100%;" border="0" width="100%" cellspacing="0" cellpadding="0" align="center">
<tbody>
<tr style="height: 107px;">
<td style="font-size: 0px; padding: 18px; height: 10px;" align="center" valign="top" bgcolor="#000034">
<div style="display: inline-block; max-width: 100%; min-width: 100px; vertical-align: top; width: 100%;">
<table style="max-width: 600px;" border="0" width="100%" cellspacing="0" cellpadding="0" align="left">
<tbody>
<tr>
<td class="mobile-center" style="font-family: Helvetica Neue, Helvetica, Arial, sans-serif; font-size: 36px; font-weight: 500; line-height: 48px;" align="left" valign="top">
<p style="font-size: 28px; font-weight: 500; margin: 0px; color: #black; text-align: center;"><span style="font-size: 12pt; color: #ffffff;">XYZ Company</span></p>
<p style="font-size: 28px; font-weight: 500; margin: 0px; color: #black; text-align: center;"><span style="font-size: 12pt; color: #ffffff;">Information Technology Services</span></p>
</td>
</tr>
</tbody>
</table>
<span style="font-size: 12pt;"></span></div>
</td>
</tr>
<tr style="height: 13px;">
<td style="padding: 20px 35px; background-color: #ffffff; text-align: left; height: 13px;" align="center" bgcolor="#ffffff">${notification:body}</td>
</tr>
<tr style="height: 268px;">
<td style="padding: 15px; background-color: #ffffff; height: 268px;" align="center" bgcolor="#ffffff">
<table style="max-width: 600px; width: 99.6479%;" border="0" width="566" cellspacing="0" cellpadding="0" align="center" height="282">
<tbody>
<tr>
<td style="font-family: Open Sans, Helvetica, Arial, sans-serif; font-size: 14px; font-weight: 400; line-height: 24px; padding: 5px 0 10px 0;" align="center">
<p><strong>Have updates for us?</strong></p>
<p>You can update your ticket by doing one of the following:</p>
<ul style="list-style-position: inside;" type="disc">
<li style="text-align: left;">From the Company <a title="Service Portal" href="sp">Service Portal</a></li>
<li style="text-align: left;">Reply to this email&nbsp;</li>
<li style="text-align: left;">Phone: 555-555-5555 , Monday to Friday 9am-6pm<br /></li>
</ul>
</td>
</tr>
<tr>
<td style="font-family: Open Sans, Helvetica, Arial, sans-serif; font-size: 14px; font-weight: 400; line-height: 24px;" align="center">
<p style="font-size: 14px; font-weight: 400; line-height: 20px; color: #777777;">This message was sent from the XYZ Company ServiceNow System</p>
</td>
</tr>
</tbody>
</table>
</td>
</tr>
</tbody>
</table>
</td>
</tr>
</tbody>
</table>
```
# Create a new template for Incident and link the layout
1. Under System Notification -> Email -> Templates, click New.
2. Title it "**Sample Incident Template**"
3. Under Email layout, Choose the "Sample Layout" we just created and save.

# Update the Incident commented for ESS to use our new template
1. Under System Notification -> Email -> Notifications, find "**Incident commented for ESS**".
2. On the tabe "What will it contain", set the  Email template field to the "Sample Incident Template" we just created.
3. (optional) Under the Message HTML, click the html code button "<>" in the layout bar and paste in the html below.

```
<div style="text-align: left;">Ticket <strong>${number}</strong> has ben updated. Please see the latest comment below.</div>
<div style="text-align: left;"><br /></div>
<div style="text-align: center;">
<table style="height: 13px; width: 100%; border-collapse: collapse; background-color: #eeeeee; border-style: none;" bgcolor="#eeeeee">
<tbody>
<tr style="height: 13px;">
<td style="width: 100%; height: 13px;">
<div style="text-align: left;">${mail_script:incident_comments}</div>
<div style="text-align: left;"><br /></div>
</td>
</tr>
</tbody>
</table>
</div>
<p><br /></p>
<center>
<table style="height: 47px;" border="0" width="181" cellspacing="0" cellpadding="0">
<tbody>
<tr style="height: 47px;">
<td style="border-radius: 5px; text-align: center; width: 177.102px; height: 47px;" align="center" bgcolor="#337ab7"><a style="font-size: 18px; font-family: Open Sans, Helvetica, Arial, sans-serif; color: #ffffff; text-decoration: none; border-radius: 2px; background-color: #337ab7; padding: 15px 30px; border: 1px solid #337ab7; display: block;" title="Service Portal" href="sp?sys_id=${sys_id}&amp;view=sp&amp;id=ticket&amp;table=incident">View Incident</a></td>
</tr>
</tbody>
</table>
</center>
```

# Example Result
![]({{ 'assets/images/template result.PNG' | relative_url }})

**Tip:**
To change the look of the text in the comments, modify the mail script **"incident_comments"**
