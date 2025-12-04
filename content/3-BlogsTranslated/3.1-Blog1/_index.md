---
title: "Blog 1"
date: "2025-09-09T15:44:00+07:00"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Supercharge Your Organization's Productivity with Amazon Q Business Browser Extension

**Authors:** Abhinand Sukumar and Firaz Akmal | **Published:** Sep 17, 2025 | **In:** Amazon Q Business, Technical How-to

---

Generative AI solutions like Amazon Q Business are transforming how employees work. Organizations across industries are embracing these tools to help their workforce extract valuable insights from increasingly fragmented data to accelerate decision-making.

However, adopting Generative AI tools is not without challenges. Two obstacles have emerged in deploying Generative AI solutions. First, users often find themselves forced to abandon familiar workflows, manually transferring data to an AI assistant for analysis. This creates unnecessary friction and increases time to value. Second, the absence of Generative AI tools in commonly used software creates difficulties for employees in identifying opportunities where AI could significantly increase their productivity.

Enter Amazon Q Business, a Generative AI-powered assistant designed specifically for the modern workplace, allowing you to engage in conversations, solve complex problems, and take action by seamlessly connecting to company data and enterprise systems. Amazon Q Business provides employees with instant access to relevant information and advice, streamlines tasks, accelerates decision-making, while driving creativity and innovation in the workplace.

Recently, we launched the Amazon Q Business browser extension in Amazon Q Business, and it's now available for Amazon Q Business subscribers (Lite and Pro). The Amazon Q Business browser extension brings the power of Amazon Q Business directly into your browser, enabling you to receive context-aware Generative AI assistance and get direct help with daily tasks.

In this post, we'll show you how to deploy this solution for your own business, providing your team with seamless access to AI-driven insights.

## Use Cases for Amazon Q Business Browser Extension

The Amazon Q Business browser extension has been deployed to all Amazonians, helping tens of thousands of users work more efficiently every day. In this section, we highlight some of the highest-impact use cases where Amazonians use the Amazon Q Business browser extension to increase productivity.

### Web Content Analysis

Business and technical teams need to analyze and synthesize information across multiple reports, competitive analyses, and industry documents found outside company data to develop insights and strategies. They must ensure their strategic recommendations are based on verified data sources and reliable industry information. Additionally, identifying patterns across multiple sources is time-consuming and complex.

With the Amazon Q Business browser extension, strategists can quickly generate industry insights and identify trends across reliable internal and external data sources in just seconds, while still maintaining the human element in strategic thinking.

Watch the following demo video:

![Web content analysis demo](https://d2908q01vomqb2.cloudfront.net/artifacts/DBSBlogs/ML-18631/browser_extension_demo_use_case_1.mp4?_=1)

### Content Quality Improvement

The Amazon Q Business browser extension offers unique capabilities to incorporate context that may not be easily available to your Generative AI assistant. You can use the Amazon Q Business browser extension to create content and improve content quality by including multiple discrete sources in your queries that are typically not available to Generative AI assistants.

You can use it to perform real-time content validation from various sources and incorporate web-based style guides and best practices to accelerate the content creation process.

Watch the following demo:

![Content quality improvement demo](https://d2908q01vomqb2.cloudfront.net/artifacts/DBSBlogs/ML-18631/browser_extension_demo_use_case_3.mp4?_=2)

## Solution Overview

In the following sections, we'll guide you on how to get started with the Amazon Q Business browser extension if you've already enabled Amazon Q Business for your organization. To learn more, see the Configuring Amazon Q Business browser extension for use section.

## Prerequisites

Complete the prerequisite steps in this section before deploying the browser extension.

### Create an Amazon Q Business Application and Subscribe Your Users

The Amazon Q Business browser extension is a feature of Amazon Q Business and requires customers to first create an Amazon Q Business application and subscribe their users before the browser extension can be enabled. To learn more about how you can get started with Amazon Q Business, see Getting started with Amazon Q Business.

### Set Up Amazon Q Business Web Experience

The extension uses the Amazon Q Business web experience client as a mechanism to authenticate users and provide Amazon Q Business features. The first step to enable the extension is to create an Amazon Q Business web experience. If you've already created a web experience for your users, you can skip this step.

However, if you've developed a custom web experience using Amazon Q Business APIs, complete the following steps to create an Amazon Q Business web experience:

1. On the Amazon Q Business console, navigate to your Amazon Q Business application.
2. The **Web experience settings** section shows whether you have a web experience deployed.
3. If you haven't deployed a web experience, this section will be empty, with the message "A web experience needs to be created before deploying."

![Empty web experience settings](https://i.imgur.com/GjQ8uK2.png)

4. At the top of your application page, choose **Edit**.

![Edit button on application page](https://i.imgur.com/w9U4nUu.png)

5. For **Outcome**, choose **Web experience**.
6. Choose **Update**.

![Select Web experience and Update](https://i.imgur.com/Q8x0WkO.png)

This step may take a few minutes to complete. After your web experience is deployed, you'll find a URL where your web experience is hosted on your Amazon Q Business application details page. Save this URL for later use.

![Web experience URL](https://i.imgur.com/R3x3W0c.png)

### Grant Users Permission to Send Queries Directly to the Large Language Model

The Amazon Q Business browser extension can incorporate the user's web page context into queries by sending web page content as an attachment along with the user's prompt. Because the attachment feature is only available for General knowledge mode, the extension requires Amazon Q Business admins to grant users permission to send queries directly to the large language model (LLM) to fully leverage the extension's feature set.

Without this prerequisite, users can only access company knowledge through the extension and cannot ask Amazon Q Business about their web page content.

Amazon Q Business does not store user conversation data and does not use queries or conversations to train its LLMs. Conversations are only stored in the application for 30 days. You can delete these conversations by accessing the Amazon Q Business web experience and choosing Chat in the navigation panel, as illustrated in the following screenshot.

![Delete conversations in web experience](https://i.imgur.com/g0t4wP9.png)

To grant users permission to send queries directly to the Amazon Q LLM, complete the following steps:

1. On the Amazon Q Business console, navigate to your application.
2. Choose **Admin controls and guardrails** in the navigation panel.

![Admin controls and guardrails](https://i.imgur.com/n7q0Yq9.png)

3. In the **Global controls** section, choose **Edit**.

![Global controls Edit](https://i.imgur.com/R0y2WqE.png)

4. Select **Allow end users to send queries directly to the LLM**.
5. Choose **Save**.

![Allow end users to send queries](https://i.imgur.com/g8v3ZqS.png)

You're now ready to enable the browser extension for your users.

## Configure Amazon Q Business Browser Extension

Now that you've completed the prerequisites for the browser extension, complete the following steps to enable the browser extension for your users:

1. On the Amazon Q Business console, navigate to your application.
2. In the **Enhancements** section in the navigation panel, choose **Integrations**.
3. In the **Browser extensions** section, choose **Edit**.

![Browser extensions Edit](https://i.imgur.com/b0y1NqW.png)

4. Select the checkboxes for the browser extensions you want to enable:
   - The **Chromium** checkbox enables the Chrome store extension, supporting Google Chrome and Microsoft Edge browsers.
   - The **Firefox** checkbox enables the Firefox Browser add-on for Firefox browsers.

5. You can also view the Chrome or Firefox store pages for the extension using the links in the corresponding **Learn more** sections.
6. Choose **Save**.

![Select Chromium and Firefox](https://i.imgur.com/h5T2J8x.png)

Your users will now see instructions for installing the Amazon Q Business browser extension the next time they log into the Amazon Q Business web experience. If you haven't already, share the web experience URL you received in previous steps with your users so they can follow the steps to install the browser extension.

### Enable the Extension If You Use IAM Federation for Amazon Q Business

If you're using an external identity provider (IdP) for your Amazon Q Business application, you must allow-list the extension with that provider before users can start using the extension.

Allow-list the following URLs with your IdP to enable the extension:

- For the Chromium browser extension (suitable for Google Chrome and Microsoft Edge): https://feihpdljijcgnokhfoibicengfiellbp.chromiumapp.org/
- For the Mozilla Firefox extension: https://ba6e8e6e4fa44c1057cf5f26fba9b2e788dfc34f.extensions.allizom.org/

You don't need to complete the above steps if using AWS IAM Identity Center as the authentication solution for your Amazon Q Business application.

## Getting Started with the Browser Extension

After you share the web experience URL with users, they can use this URL to find the extension store page and install it. Users complete the following steps:

1. Log into the Amazon Q Business web experience provided by your administrator.
2. You'll see a banner announcing that the administrator has enabled the extension for you.
3. Choose **Install extension**.

![Install extension banner](https://i.imgur.com/f7j8bQ6.png)

4. The link will take you to the appropriate store page for the Amazon Q Business extension based on your browser.
5. Choose **Add to Chrome** or the corresponding installation option for your browser.

![Chrome web store](https://i.imgur.com/b0x8iYg.png)

6. After installation, you'll see the extension in your browser toolbar, under Extensions.
7. You can choose the pin icon to pin the extension.

![Pin extension](https://i.imgur.com/t3w7cM7.png)

8. When opening the extension, you'll see a side pane as illustrated.
9. The extension will automatically detect the correct web experience URL from open tabs to help you sign in.
10. If not, enter the web experience URL provided by your administrator in the **Amazon Q URL** field and choose **Sign in**.

![Sign in pane](https://i.imgur.com/x4W2bQp.png)

After signing in, you're ready! Refer to the previous section on Amazon use cases for inspiration on using the extension to increase productivity.

## Deploy Amazon Q Business Extension on Behalf of Users

Some administrators may choose to directly deploy the Amazon Q Business extension to user browsers to streamline and accelerate adoption. Businesses use different mobile device management (MDM) software and have different requirements for their browser policies.

To deploy the Amazon Q Business extension, refer to the following resources:

- Mozilla Firefox: policy settings
- Google Chrome: policy settings
- Microsoft Edge:
  - Policy settings
  - Reference guide

## Customize Amazon Q Business Extension for Your Business

Some administrators may choose to customize the appearance of the Amazon Q Business extension to suit business needs. This section outlines the supported customization functionality of the extension and corresponding browser extension policy values to configure on user browsers.

### Remove Amazon Q Business URL Input Field from Browser Extension Sign-in Page

If you don't want to require users to enter the Amazon Q Business web experience URL when signing in, you can set the default URL on their behalf by setting the `Q_BIZ_BROWSER_EXTENSION_URL` policy to the appropriate Amazon Q Business web experience URL for your users.

![Customize sign-in URL](https://i.imgur.com/C3p9XjG.png)

### Replace Browser Extension Toolbar Icon

You can modify the browser extension toolbar icon by setting the value of one or more of the following browser policy keys to your PNG or SVG image URL or a valid datauri for your users:

- `Q_BIZ_BROWSER_EXTENSION_ICON_128` (required)
- `Q_BIZ_BROWSER_EXTENSION_ICON_16` (optional)
- `Q_BIZ_BROWSER_EXTENSION_ICON_32` (optional)
- `Q_BIZ_BROWSER_EXTENSION_ICON_48` (optional)

![Replace toolbar icon](https://i.imgur.com/placeholder.png)

### Replace Logo or Icon in Browser Extension Window

To change the logo or icon in your browser extension window, set the value of the `Q_BIZ_BROWSER_EXTENSION_LOGO` policy key with a URL to your PNG or SVG image or a valid datauri for your users.

![Replace logo](https://i.imgur.com/placeholder2.png)

### Modify Browser Extension Name Displayed in Browser Extension Window

To replace references to "Amazon Q", "Amazon Q Business", "AWS" and "Amazon Web Services" with a name of your choice in the browser extension window, set the value of the `Q_BIZ_BROWSER_EXTENSION_ENTERPRISE_NAME` policy key with the new name for your users.

![Modify extension name](https://i.imgur.com/placeholder3.png)

### Modify Browser Extension Title in Hover Text

To change the browser extension title as it appears in hover text over your extension ("Amazon Q Business has access to this site", as seen in previous screenshot), set the `Q_BIZ_BROWSER_EXTENSION_TITLE_NAME` policy to the appropriate string for your users.

![Modify hover text](https://i.imgur.com/placeholder4.png)

### Replace AI Policy Link in Browser Extension Footer with Your Own Link

To replace the link text in your browser extension footer, set `Q_BIZ_BROWSER_EXTENSION_FOOTER_POLICY_NAME` to the appropriate string for your users.

To replace the URL in your browser extension footer, set `Q_BIZ_BROWSER_EXTENSION_FOOTER_POLICY_URL` to the appropriate URL for your users.

![Replace footer link](https://i.imgur.com/placeholder5.png)

Congratulations! You and your organization are ready to receive generative assistance for your browser-based tasks.

## Cleanup

This section outlines steps to disable or remove the browser extension or undo deployments and customizations for your users.

### Disable Amazon Q Business Browser Extension via Amazon Q Business Console

You can disable the Amazon Q Business browser extension from the Amazon Q Business console at any time, even before removing the browser extension from your users' browsers. To do so, complete the following steps:

1. On the Amazon Q Business console, navigate to your application.
2. Under Enhancements in the navigation pane, choose Integrations.
3. In the Browser extensions section, choose Edit.

![Disable extension step](https://i.imgur.com/placeholder6.png)

4. Deselect the checkboxes for the browser extensions you want to disable:
   - The **Chromium** checkbox disables the Chrome store extension, supporting Google Chrome and Microsoft Edge browsers.
   - The **Firefox** checkbox disables the Firefox Browser add-on for Firefox browsers.

5. Choose **Save**.

![Save disable](https://i.imgur.com/placeholder7.png)

### Undo Deployment of Amazon Q Business Browser Extension on Behalf of Users

Enterprises use various mobile device management software and have different requirements for their browser policies. If you deployed the browser extension by updating browser policy settings, you should remove those policies by following instructions in the policy settings documentation for corresponding browsers:

- Mozilla Firefox policy settings
- Google Chrome policy settings
- Microsoft Edge:
  - Policy settings
  - Reference guide

### Undo Customization of Amazon Q Business Browser Extension

If you customized the Amazon Q Business browser extension by editing browser policies as detailed earlier in this post, you can undo those customizations by simply removing the corresponding policy entry in your browser policy settings.

## Conclusion

In this post, we showed you how to use the Amazon Q Business browser extension to provide your team with seamless access to AI-driven insights and assistance. The browser extension is now available in AWS Regions US East (N. Virginia) and US West (Oregon) for Mozilla, Google Chrome, and Microsoft Edge as part of the Lite Subscription. There is no additional cost to use the browser extension.

To get started, log into the Amazon Q Business console and set up the browser extension for your Amazon Q Business application. To learn more, see Configuring the Amazon Q Business browser extension for use.

---

## About the Authors

**Firaz Akmal** is a Sr. Product Manager for Amazon Q Business and has been with AWS for over 8 years. He is a customer advocate, helping customers transform search and generative AI use-cases on AWS. Outside of work, Firaz enjoys spending time in the PNW mountains or experiencing the world through his daughter's perspective.

**Abhinand Sukumar** is a Senior Product Manager at Amazon Web Services for Amazon Q Business, where he drives product vision and roadmap for innovative generative AI solutions. Abhinand works closely with customers and engineering to deliver successful integrations, including the browser extension. His expertise spans generative AI experiences and AI/ML educational devices, with a deep passion for education, artificial intelligence, and design thinking. Before joining AWS, Abhinand worked as an embedded software engineer in the networking industry, with 5-6 years of experience in technology.

