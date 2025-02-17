专为渗透测试者设计的网络钓鱼平台。该工具使我们能够在 Outlook 中制作网络钓鱼电子邮件、快速克隆它们、自动为其设置模板以进行大规模分发、测试电子邮件模板、安排网络钓鱼活动并跟踪网络钓鱼结果。

与流行的网络钓鱼框架（如 GoPhish）相比，它允许对 SMTP 和邮件标头进行更精细的控制，允许直接服务器到服务器 SMTP，支持 DMARC 和 DKIM，并且可以使用 websockets 显示实时结果，而无需刷新结果页
## Phishmonger

Phishing platform designed for pentesters. This tool allows us to craft phishing emails in Outlook, clone them quickly, automatically template them for mass distribution, test email templates, schedule phishing campaigns, and track phishing results. 

Compared with popular phishing frameworks like GoPhish, it allows for more granular control over SMTP and mail headers, allows direct server-to-server SMTP, supports DMARC and DKIM, and can show real-time results using websockets instead of needing to refresh the results page.

Installation 
=====

Go check out [Flik](https://github.com/fkasler/flik) to automate Phishmonger setup. Get a Gandi.net account to make setup a breeze.

Getting Started
=====

- Start the server in a screen/tmux and make sure it runs without errors:

```
cd tools/phishmonger
node index.js
```

Usage
=====

- Phishmonger is not just GoPhish in Node! You do not have to set up a separate mail server. Phishmonger itself is a mail server. You can, and often might want to send directly to your target's mail server(s). This opens up the full range of spoofing techniques and provides granular control over the SMTP protocol and message content. This design is intentional and is meant to familiarize operators with SMTP and MIME while making modifications as simple as possible.

- Start by using the "Create Campaign" option. You can capture emails and save as templates or make modifications and save directly as campaigns. If you have templates, select your template and use the "Campaign from Template" option to shortcut the process.

- Click the "Capture Email" button in order to start a listener on your phishmonger's port 25. The button should turn grey to let you know it is working.

- You can now draft your phishing email in Outlook and send to your phishmonger domain:

```
To: anything@myphishmongerdomain.com
```

- Once the email is captured by the server, Phishmonger will automatically parse out the email sections in the web GUI.

- There are several buttons that can be used to help the templating process. 
  - Reset Captured Email: resets all changes made to the captured email. Useful if you make a mistake but don't want to resend the email.
  - RFC Only Headers: trims out unnecessary SMTP headers so you can start with only the sections you need.
  - Preview Email: Lets you see a markup of the message so you can see how any modifications will look.
  - Find & Replace: Makes a global replacement on every email section and header.

- Some of these buttons only perform actions on the email content directly below them:
  - Base64 Decode: Decodes the section if it was Base64 encoded
  - Quoted Printable Decode: Decodes QP text. You can tell it is QP if every line of text ends with =
  - Pretty Print: Indents HTML content sections to make them more human-friendly
  - Attach Images: Works on HTML content sections to swap out external image "src" attributes with a CID to an attached image and adds the base64 of the attached image to the email
  - Preview Image: Works on Image content sections and allows you to make sure the image looks the way you want. THIS DOES NOT SHOW YOU A PREVIEW OF THE EMAIL, just a single image attachment

- Several of the options in the toolbar on the right are designed as string substitutions. For example, SuppliedPhishingLink will be replaced by Phishmonger with the URL of your phishing domain:

```
Click <a href="SuppliedPhishingLink">here</a> to download my malware.
```
- Other string substitutions like SuppliedFirstName will be replaced my Phishmonger based on the provided targets list

- You can use the "Send Test" button to test out the template and mail server settings. I recommend testing against your own inbox, making any needed tweaks, and then test against [mail-tester](https://www.mail-tester.com/) to see if you have any major red flags.

- You can use "Save as Template" in order to save a generic template for fast re-creation later, or use "Save as Campaign" to create a one-off campaign. In either case, you should see a prompt from the server when it has saved your work.

- Navigate "Back to Campaigns" to set up a target list, make any final tweaks to the email, and schedule/run the campaign.

- To schedule a campaign, click on the campaign name from the /admin URL, Add/Modify your targets list, and either use the "Send Campaign" button to start the campaign immediately or modify the time next to the "Schedule Campaign" button and click the "Schedule Campaign" button after you have updated the time you would like the campaign to start. You must schedule based on the timezone of the server. You can see the current time in your server's time zone by refreshing the page. The timestamp in the input field should update to the current time.

- Most campaign events are generated by Humble Chameleon as a payload delivery and credential/session harvesting server. The benefits of this approach are that we can hide our phishing domain in order to keep it off blacklists, we can field multiple domains with the same Humble Chameleon server, modify scenarios and targets quickly, 'clone' websites on the fly, and attack 2FA protected logins all with the same setup. Configure your Humble Chameleon domain to log events to the /create_event URL on your Phishmonger server:

```
{
    "myphishingdomain.com": {
        "primary_target": "something_to_hide_behind.org",
        "secondary_target": "real_target_domain.com",
        "search_string": "document_id",
        "wwwroot": "hr_documents",
        "tracking_cookie": "evil_cookie",
        "replacements": {},
        "custom_headers": {},
        "snitch": {
            "snitch_string": "Logoff",
            "redirect_url": "https://vpn.real_target_domain.com/index.html"
        },
        "logging_endpoint": {
            "host": "www.myphishmongerdomain.com",
            "url": "/create_event",
            "auth_cookie": "admin_cookie=myadmincookievalue"
        }
    }... other phishing domains...
}
```
- All events from a campaign are pushed to the browser in real-time using websockets so you never need to refresh the page to see what is going on with your campaign. You can click on the graphs to view events that make up the data in the graphs and search through event logs

