---
title: Exploiting SSRF like a Boss — Escalation of an SSRF to Local File Read!
subtitle: Hi Guys!
Greetings everyone! Today I am doing another write-up about one of my best findings. It’s an SSRF — Server Side Request Forgery vulnerability I discovered in a private program.

# Summary for listings and search engines
summary: Hi Guys!
Greetings everyone! Today I am doing another write-up about one of my best findings. It’s an SSRF — Server Side Request Forgery vulnerability I discovered in a private program.
In the scope page, the program had few IPs with only Server-Side bugs acceptable in its scope. I picked an IP and started my recon process on it. I found a subdomain https://help.redacted.com hosted on that IP through Reverse IP scan. I used HackerTarget’s Reverse IP lookup tool “http://api.hackertarget.com/reverseiplookup/?q={IP}”
The subdomain was running a Jira instance. I quickly remembered the 
Alyssa Herrera
’s article about SSRF Exploitation in Jira instances. I checked the version of the Jira and it seems vulnerable to the SSRF.
https://help.redacted.com/plugins/servlet/oauth/users/icon-uri?consumerUri=http://google.com

Now I was able to render any webpage through that vulnerable endpoint or I could have converted it into XSS by loading an external page but as I have told earlier that the company was only interested in Server-Side and network related issues so I had to dig more.
None of the protocols i.e. gopher://, file://, ldap:// or ftp:// were working except http://. It was an azure instance so I tried to fetch the metadata file of the instance from the uri: https://help.redacted.com/plugins/servlet/oauth/users/icon-uri?consumerUri=http://169.254.169.254/metadata/v1/maintenance but I got nothing but a blank JSON response.

Now I needed to think out of the box. I started playing with the endpoint. I tried to load localhost (127.0.0.1) from the uri and it loaded the main page. An idea clicked in my mind that there might be some services were left running on internal ports. I started by the most common port 8080. And then as I thought, I was welcomed by GlassFish server’s default page.

After seeing that I quickly reached to the default administration console at Port 4848. It was the login page of GlassFish server.

I searched for the GlassFish exploits and hopefully, I found a GET based exploit: “https://www.exploit-db.com/exploits/39241/”

I crafted that payload:
https://help.redacted.com/plugins/servlet/oauth/users/icon-uri?consumerUri=http://127.0.0.1:4848/theme/META-INF/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/%c0%ae%c0%ae/etc/passwd
But it didn't work!!

 got really disappointed and I was about to give up. But then I just noticed that there is url-encoding in that exploit and when it is passed through the browser it gets decoded so I may need to double encode it to pass the correct request on to the GlassFish server.
And our final payload was:
https://help.redacted.com/plugins/servlet/oauth/users/icon-uri?consumerUri=http://127.0.0.1:4848/theme/META-INF%2f%25c0%25ae%25c0%25ae%2f%25c0%25ae%25c0%25ae%2f%25c0%25ae%25c0%25ae%2f%25c0%25ae%25c0%25ae%2f%25c0%25ae%25c0%25ae%2f%25c0%25ae%25c0%25ae%2f%25c0%25ae%25c0%25ae%2f%25c0%25ae%25c0%25ae%2f%25c0%25ae%25c0%25ae%2f%25c0%25ae%25c0%25ae%2fetc%2fpasswd

And resultantly, this simple HTTP-Protocol based SSRF was escalated to a local file read by exploiting an internal service.
# Link this post with a project
projects: []

# Date published
date: "2021-03-13T00:00:00Z"

# Date updated
lastmod: "2021-03-26T00:00:00Z"

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
image:
  caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/CpkOjOcXdUY)'
  focal_point: ""
  placement: 2
  preview_only: false

authors:
- admin
- 吳恩達

tags:
- Academic
- 开源

categories:
- Demo
- 教程
---

## Overview

1. The Wowchemy website builder for Hugo, along with its starter templates, is designed for professional creators, educators, and teams/organizations - although it can be used to create any kind of site
2. The template can be modified and customised to suit your needs. It's a good platform for anyone looking to take control of their data and online identity whilst having the convenience to start off with a **no-code solution (write in Markdown and customize with YAML parameters)** and having **flexibility to later add even deeper personalization with HTML and CSS**
3. You can work with all your favourite tools and apps with hundreds of plugins and integrations to speed up your workflows, interact with your readers, and much more

{{< figure src="https://raw.githubusercontent.com/wowchemy/wowchemy-hugo-modules/master/academic.png" title="The template is mobile first with a responsive design to ensure that your site looks stunning on every device." >}}

## Get Started

- 👉 [**Create a new site**](https://wowchemy.com/templates/)
- 📚 [**Personalize your site**](https://wowchemy.com/docs/)
- 💬 [Chat with the **Wowchemy community**](https://discord.gg/z8wNYzb) or [**Hugo community**](https://discourse.gohugo.io)
- 🐦 Twitter: [@wowchemy](https://twitter.com/wowchemy) [@GeorgeCushen](https://twitter.com/GeorgeCushen) [#MadeWithWowchemy](https://twitter.com/search?q=(%23MadeWithWowchemy%20OR%20%23MadeWithAcademic)&src=typed_query)
- 💡 [Request a **feature** or report a **bug** for _Wowchemy_](https://github.com/wowchemy/wowchemy-hugo-modules/issues)
- ⬆️ **Updating Wowchemy?** View the [Update Guide](https://wowchemy.com/docs/guide/update/) and [Release Notes](https://wowchemy.com/updates/)

## Crowd-funded open-source software

To help us develop this template and software sustainably under the MIT license, we ask all individuals and businesses that use it to help support its ongoing maintenance and development via sponsorship.

### [❤️ Click here to become a sponsor and help support Wowchemy's future ❤️](https://wowchemy.com/plans/)

As a token of appreciation for sponsoring, you can **unlock [these](https://wowchemy.com/plans/) awesome rewards and extra features 🦄✨**

## Ecosystem

* **[Hugo Academic CLI](https://github.com/wowchemy/hugo-academic-cli):** Automatically import publications from BibTeX

## Inspiration

[Check out the latest **demo**](https://academic-demo.netlify.com/) of what you'll get in less than 10 minutes, or [view the **showcase**](https://wowchemy.com/user-stories/) of personal, project, and business sites.

## Features

- **Page builder** - Create *anything* with [**widgets**](https://wowchemy.com/docs/page-builder/) and [**elements**](https://wowchemy.com/docs/writing-markdown-latex/)
- **Edit any type of content** - Blog posts, publications, talks, slides, projects, and more!
- **Create content** in [**Markdown**](https://wowchemy.com/docs/writing-markdown-latex/), [**Jupyter**](https://wowchemy.com/docs/import/jupyter/), or [**RStudio**](https://wowchemy.com/docs/install-locally/)
- **Plugin System** - Fully customizable [**color** and **font themes**](https://wowchemy.com/docs/customization/)
- **Display Code and Math** - Code highlighting and [LaTeX math](https://en.wikibooks.org/wiki/LaTeX/Mathematics) supported
- **Integrations** - [Google Analytics](https://analytics.google.com), [Disqus commenting](https://disqus.com), Maps, Contact Forms, and more!
- **Beautiful Site** - Simple and refreshing one page design
- **Industry-Leading SEO** - Help get your website found on search engines and social media
- **Media Galleries** - Display your images and videos with captions in a customizable gallery
- **Mobile Friendly** - Look amazing on every screen with a mobile friendly version of your site
- **Multi-language** - 34+ language packs including English, 中文, and Português
- **Multi-user** - Each author gets their own profile page
- **Privacy Pack** - Assists with GDPR
- **Stand Out** - Bring your site to life with animation, parallax backgrounds, and scroll effects
- **One-Click Deployment** - No servers. No databases. Only files.

## Themes

Wowchemy and its templates come with **automatic day (light) and night (dark) mode** built-in. Alternatively, visitors can choose their preferred mode - click the moon icon in the top right of the [Demo](https://academic-demo.netlify.com/) to see it in action! Day/night mode can also be disabled by the site admin in `params.toml`.

[Choose a stunning **theme** and **font**](https://wowchemy.com/docs/customization) for your site. Themes are fully customizable.

## License

Copyright 2016-present [George Cushen](https://georgecushen.com).

Released under the [MIT](https://github.com/wowchemy/wowchemy-hugo-modules/blob/master/LICENSE.md) license.
