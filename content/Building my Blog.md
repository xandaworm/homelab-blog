---
publish: true
title: Building my Blog
created: 2026-10-08T21:54:46.085Z
modified: 2026-10-13T16:57:27.999Z
tags:
  - homelab
  - syncthing
  - quartz
  - github
  - cloudflare
---

**A guide for seamless Obsidian sync using Syncthing and serving notes over the web via Quartz, Quartz Syncer, and Cloudflare Pages**

I created this blog with the intention of showing off my homelab and other tech projects, but the idea to create it came from an entirely unrelated problem.

# The problem

In my free-time, I really enjoy playing card games, board games, and role-playing games with my friends and girlfriend. Every Sunday we all meet to play Dungeon Crawl Classics (DCC), a re-imagining of old-school Dungeons & Dragons with a modernized ruleset. I write and run our adventures, which has turned into a creative writing exercise for me, and I built my own fantasy world with politics, history, and characters. Because we only play on a weekly basis, sometimes missing a session due to scheduling conflicts, my friends are inclined to forget things that happen. Some of them take notes, but even then, not everything of importance gets written down. Since I run and write the adventures, I keep track of everything that goes on, which is in an [Obsidian](https://obsidian.md/) vault. I cannot share the full vault of notes with everyone, as it includes major game spoilers. It can also be inconvenient to search my notes for something specific that my friends want reminded of, interrupting the flow of the game. So I wanted to find a way to serve specific game notes to my friends that is easily searchable and always available.

# The solution

I had a conversation with a friend from college about how he hosts a wiki for his homelab documentation and college notes. He mentioned using [Quartz](https://quartz.jzhao.xyz/) , a static site generator that converts Markdown. From there it clicked, I already use Obsidian to keep notes (which uses Markdown), why don't I just make a wiki for my players?
[So I did](https://wiki.blueinblueeyes.org/) (it is a little barebones at the time of writing, I plan to add more) and I also made this blog too! How did I do it?

# Choosing my tools

First, I chose what tools I would use. The first thing I wanted to do was find some way to build an ecosystem where all my Obsidian vaults can seamlessly sync such that I can access them from anywhere on any device. This was something I was meaning to do for a while, and building this drove that. To do this I chose [Syncthing](https://syncthing.net/) as my tool of choice. I considered other options, like Obsidian's own sync service, iCloud, OneDrive, but I did not choose these for two reasons: There was no point in paying for such a service when I can host it myself and store the folders on my NAS, and digital sovereignty. I also looked into other self-hosted options, but Syncthing has a large community, with much support, with built-in TLS encryption and authentication.

I chose Quartz as my static site generator per my friend's recommendation. I also looked into other static site generators, like [Jekyll](https://jekyllrb.com/). Jekyll is far more flexible from a UI perspective with a larger community, with more professional usage than Quartz. It even powers GitHub pages. I ultimately went with Quartz because it has a much simpler out of the box experience, and is specifically designed for usage with Obsidian vaults, so there is a near 0 amount of tinkering to have the two work together. Eventually, I'd like to move this blog over to Jekyll so I can use its extra UI customization power and plethora of community plugins. That's not to say Quartz doesn't have UI customization or plugins, but doing anything out of Quartz's mold typically requires writing custom TypeScript, JSX, or CSS as the plugin community is younger and less in quantity.
Essentially if you want to quickly create a website using your Obsidian markdown notes, use Quartz. If you want more plugins for customization, use Jekyll, at the expense of spending a little more time to get things up and running to work with your Obsidian vault and building a pretty UI.

To host my webpages, I decided to use Cloudflare pages as I am well familiar with the platform as they are my domain provider. Cloudflare pages also supports GitHub repositories, and will update your hosted page on github commits. The other alternatives I considered were self-hosting and using a tool like nginx to host, as well as GitHub pages. Lastly, the free tier of Cloudflare pages provides more than enough support for this project. I opted out of self-hosting, as it opens up security concerns that I did not believe were necessary for the sake of digital sovereignty. I chose Cloudflare pages over GitHub pages due to my familiarity with the former.

# Building the Blog

Below is a step-by-step of how I configured my platform. Refer to these steps as well as the [[#Sources]] I provided to help you recreate this.

## (OPTIONAL) Syncthing Configuration

The first task to complete was the deployment of Syncthing on my server. This ensures that there was an always online peer (as Syncthing works in a peer-to-peer manner) for my Obsidian vaults to sync to. Note that Syncthing is optional but it provides an additional layer of data redundancy and the ability to work on my blog from multiple devices seamlessly. I used docker for my server, but you can install and configure Syncthing in your own way.

1. Created a folder on my NAS to store my Obsidian vaults
2. Deployed Syncthing using the following docker compose

`---`
`services:`
`syncthing:`
`image: lscr.io/linuxserver/syncthing:latest`
`container_name: syncthing`
`hostname: syncthing #optional`
`environment:`
`- PUID=1000`
`- PGID=1000`
`- TZ=Etc/UTC`
`volumes:`
`- ./config:/config`
`- /your/obsidian/directory/path/here:/obsidian`
`ports:`
`- 8384:8384`
`- 22000:22000/tcp`
`- 22000:22000/udp`
`- 21027:21027/udp`
`restart: unless-stopped`

3. Navigated to `<docker-host-ip>:8384` in a web browser
4. Clicked **Add Folder**
5. Entered `/obsidian` under _Folder Path_
6. Entered `Obsidian` under _Folder Name_
7. OPTIONAL: Clicked the **File Versioning** tab and selected _Staggered File Versioning_
   1. This adds versioning in the following way: Files are moved to date stamped versions in a .stversions directory when replaced or deleted by Syncthing. Versions are automatically deleted if they are older than the maximum age or exceed the number of files allowed in an interval. The following intervals are used: for the first hour a version is kept every 30 seconds, for the first day a version is kept every hour, for the first 30 days a version is kept every day, until the maximum age a version is kept every week.
8. Clicked **Save**

The next task was to add a client device into my Syncthing ecosystem. In order to do this, I needed to install the service on my device with the [Syncthing installer](https://syncthing.net/downloads/). In my case, this was a Windows device, so I used the Windows installer. After installation, I did the following to sync my client device with the server, on the client device:

1. Navigated to `localhost:8384` in a web browser
2. Clicked **Add Remote Device**
3. Entered `<device-id>` under _Device ID_
   1. To obtain the server device ID, I navigated to my server's syncthing at `<docker-host-ip>:8384`
   2. Clicked **Actions** | **Show ID**
   3. Clicked **Copy**
4. Navigated to `<docker-host-ip>:8384`
5. Accepted the pop-up to allow my client device to join the Syncthing Environment
6. Under _Folders_, Clicked **Obsidian** | **Edit** | **Sharing**
7. Ticked the check-box next to my client device
8. Clicked **Save**
9. Navigated to `localhost:8384`
10. Accepted the pop-up to share the Obsidian folder
11. Clicked **Obsidian** | **Edit**
12. Entered `<client/obsidian/vaults/directory/here>` under _Folder Path_
13. Clicked **Save**

Now your syncthing should be working! Optionally, you can set up a username and password for each of your syncthing instances, which I highly recommend. Here's what I did to set that up:

1. Navigated to `<syncthing-instance-ip>:8384`
2. Clicked **Actions** | **Settings** | **GUI**
3. Entered `<username>` under _GUI Authentication User_
4. Entered `<password>` under _GUI Authentication Password_
5. Clicked **Save**

Syncthing should then sign you out, and you will be prompted to login with the credentials you entered.

## Quartz Configuration

In order to use Quartz with Github or Cloudflare pages, you'll want to create a GitHub repository for your site. Below I will explain how I did that, with the assumption that you already have a GitHub account, which is necessary for this part.

1. Navigated to the [Quartz repository](https://github.com/jackyzha0/quartz)
2. Clicked **Use this template** | **Create a new repository**
3. Entered `<repository-name>` in _Repository name_
4. Optionally added a description and privacy configuration
5. Clicked **Create repository**

Once you have the repository created, you can get to work. I will be working in an Ubuntu CLI with git for this. In order to use git with GitHub, you'll need to setup authentication if you haven't already. You may follow what I did below to do so, starting from your Linux CLI.

1. Entered `git config --global user.name <your-github-username>`
2. Entered `git config --global user.email <your-github-email>`
3. Entered `ssh-keygen -t ed25519 -C "<your-label>"`
4. Entered `cat ~/.ssh/id_ed25519.pub`
5. Copied the printed key, which started with _ssh-ed25519_
6. Navigated to [GitHub](https://github.com) in a web browser
7. Clicked **Profile Icon** | **Settings** | **SSH and GPG keys** | **New SSH key**
8. Entered `<your-label>` under _Title_
9. Pasted the copied key under _Key_
10. Clicked **Add SSH key**
11. In my Linux CLI, entered `ssh -T git@github.com`

With the last step, you should see a welcome message with your username stating that you have successfully authenticated. You generated an SSH keypair with the ed25519 algorithm, the current standard for SSH keypairs. Now you can use git with GitHub. You'll also want to ensure that you have at least Node v22 and npm 10.9.2. Check these with `node -v`and `npm -v` respectively. If you do not have these, recreate what I did below in your Linux CLI.

1. Entered `curl -fsSL https://deb.nodesource.com/setup_24.x | sudo -E bash -`
2. Entered `sudo apt install -y nodejs`

You should receive some sort of success message. Run `node -v`and `npm -v` again, and make sure their version numbers are higher than v22 and 10.9.2 respectively. Now you can initialize your Quartz site, in your Linux CLI.

1. Navigated to my working directory
2. Entered `git clone https://github.com/<your-username>/<your-github-quartz-repository>.git`
3. Entered `cd <your-github-quartz-repository>`
4. Entered `npm i`
5. Entered `npx quartz create`
6. Selected an applicable template, blog in my case
7. Selected new for the content
8. Entered `<your-website-url>` (Exclude https://)
9. Selected shortest for resolving internal links (This does not show if using the obsidian or TTRPG template)
10. Entered `npx quartz plugin install --from-config`
11. Entered `npx quartz plugin install --latest`
12. Entered `npx quartz build --serve --port:<your-port>`

If successful, you should be able to view a preview build of your website. Navigate to `<host-ip>:<your-port>` in a web browser. When you are ready to exit the preview build, press **CTRL+C** in your Linux CLI.

Lastly, we want to push our built quartz site to GitHub. Enter `npx quartz sync --no-pull` to do so. After this, if you are pushing changes, you just need to enter `npx quartz sync` from your working directory.

## Quartz Syncer Configuration

Now that you have a Quartz website established inside a GitHub repository, you'll want to set up a method to push Obsidian notes directly to your repository.  [Quartz Syncer](https://community.obsidian.md/plugins/quartz-syncer), a plugin for Obsidian, does just that. If you have Syncthing set up, note that Obsidian stores plugins and plugin information in each vault, so you will only have to configure this once. In order to configure Quartz Syncer, recreate what I did below.

1. In Obsidian, open my working vault.
2. Clicked the **Settings Gear** | **Community Plugins**
3. If you have not done so already, disable restricted mode by clicking **Exit restricted mode**
4. Clicked **Browse**
5. Searched for `"Quartz Syncer"` and selected it
6. Clicked **Install** | **Enable** | **Options** | **Run setup wizard**
7. Clicked **Connect an existing repository**
8. Navigated to [Github personal access token creation](https://github.com/settings/tokens/new?scopes=repo,workflow\&description=Quartz+Syncer) using the provided link
9. Entered `<your-token-name>` under _Note_
10. Selected an expiration date under _Expiration_. When the token expires you will have to generate a new one. From a security standpoint, you'll want to use a 30 day expiration, but you will have to update Quartz Syncer with a new token on expiration.
11. Scrolled down and clicked **Generate token**
12. Copy the provided token. Note that you will not be able to obtain this token again, so be sure to copy it now
13. In the Quartz Syncer setup wizard, pasted the token into the text bar labeled _GitHub personal access token_
14. Clicked **Validate**
15. Selected the applicable repository
16. Clicked **Continue**
17. Verify you have the correct repository and branch. Clicked **Save settings**

Now your Quartz Syncer is configured! In order to use it, you will want to mark pages that you want to publish using file properties.

1. Navigated to a note I want to publish in Obsidian
2. Clicked the **Three dots icon** | **Add file property**
3. Typed `publish` in the left box, and `true` in the right box

In order to publish:

1. Clicked the **Green leaf icon** (bottom right of note)
2. Ticked the _UNPUBLISHED_ box
3. Clicked **Publish to remote**

From there, Quartz Syncer will add your page right into your Quartz repository. If you have GitHub Pages or Cloudflare Pages configured, your website will be rebuilt with the newly published note. Speaking of Cloudflare pages...

## Cloudflare Pages Configuration

I will assume you already have a Cloudflare account set up, if you do not, create one. Start by navigating to your [Cloudflare dashboard](https://dash.cloudflare.com).

1. Clicked **Compute** | **Workers & Pages** | **Create Application** | **Continue with GitHub**
2. Selected my GitHub account (Add yours here, if you haven't already by clicking **+ Add GitHub Account**, select your repository access settings, and click **Save**)
3. Selected your Quartz repository
4. Entered `git fetch --unshallow && npx quartz plugin install && npx quartz build` under _Build command_. Note that `git fetch --unshallow` is only necessary if your site is using timestamps.
5. Clicked **Deploy**

Now your site should deploy with a provided domain by Cloudflare. If you own a domain, you can add it. Recreate my steps from the Cloudflare dashboard to do so.

1. Clicked **Compute** | **Workers & Pages**
2. Clicked my site
3. Clicked **Custom domains** | **Set up a custom domain**
4. Entered `<your-website-url>` under _Domain_
5. Clicked **Continue** | **Activate domain**

Your site should now be viewable directly from your URL.

## Comments & Plugins Configuration

As you can see, my site has a comments feature. This comes pre-installed and enabled with the Quartz blog template, so I opted to configure it. The comments feature is a community plugin featured in Quartz. Plugins can be disabled, enabled, and modified in your repository in the `quartz.config.yaml` file. See the Quartz documentation for the [list of community plugins](https://quartz.jzhao.xyz/plugins/) and [plugin configuration](https://quartz.jzhao.xyz/configuration#plugins) for more information. Below, I will recreate how I created a working comment section for my blog, using the [Quartz comments plugin](https://quartz.jzhao.xyz/features/comments) with [giscus.](https://giscus.app/)

1. In GitHub, clicked **+** | **New Repository**
2. Entered `<your-discussion-repo-name>` under _Repository name_
3. Optionally, give it a description
4. Selected _Public_ under _Choose visibility_ (Your repo must be public for this to work)
5. Clicked **Create repository**

This creates a repository that will be used with giscus as a commenting platform for your website.
Next, you'll need to configure the giscus app for access to your repository.

1. Navigated to the [giscus github apps page](https://github.com/apps/giscus)
2. Clicked **Configure**
3. Selected _Only select repositories_ under _Repository access_
4. Clicked **Select repositories**
5. Selected the discussion repository
6. Clicked **Save**

Now you have giscus added to your discussion repository. Next you will need to enable GitHub discussions for your discussion repository.

1. Navigated to GitHub discussion repository
2. Clicked **Settings**
3. Ticked the _Discussions_ check box

Now that you have GitHub discussions enabled with giscus access, you will need to edit your `quartz.config.yaml` to add comments support. To figure out properties to edit, you'll need to head to the [giscus app page](https://giscus.app/).

1. On the giscus app page, entered `<your-discussion-repository>` in the _repository:_ text box
2. Selected _Announcements_ under _Discussion Category_
3. On your working Linux CLI, navigated to the Quartz repository
4. Entered `nano quartz.config.yaml`
5. Found the comments plugin under  ` - source: "@quartz-community/comments"`
6. Changed the following under the plugin, according to the giscus app page output:
   `- source: "@quartz-community/comments"`
   `enabled: true`
   `options:`
   `provider: giscus`
   `options:`
   `repo: <your-discussion-repo>
        `repoId: <your-repo-id>
   `category: Announcements`
   `categoryId: <your-category-id>
        `mapping: pathname`
        `strict: false`
        `reactionsEnabled: true`
        `inputPosition: bottom`
        `lightTheme: light`
        `darkTheme: dark`
        `lang: en`The fields for your repo id and category id will be generated on the giscus app page under *Enable giscus*. Use the fields`data-repo-id`and`data-category-id\` for those fields respectively.
7. Pressed **CTRL+X** | **Y**
8. Entered `npx quartz sync`

## Configuring index.md and final touches

`index.md` serves as the homepage to your Quartz site. When you go to build a Quartz site using the new content option, it will create one automatically for you. index.md can be edited in multiple ways. You can edit directly from your GitHub or cloned git repositories, (using `npx quartz sync` with the latter to push it), or copy it to your Obsidian vault and edit and publish it from there. Keep in mind `index.md` will always be your homepage. It is located under the `content` directory of your repository. Now that you have your site set up and running, you can use the myriad of [[#Sources]] provided to further customize and improve your site.

# Troubleshooting

Below are some issues I ran into and how I fixed them. I will update these as more arise.

## 404 Page and tag index under recent posts

One of the default enabled plugins, recent-notes, will provide a feed of recent posts. On default, it shows the 5 most recently posted notes. Quartz generates a tag index and 404 page on build, which end up being included on the recent notes. Additionally, when new tags are used, new tag pages are created. Ultimately, I did not want these to show while I have less than 5 pages, and did not want to limit recent posts to less than 5.
To fix the tag index and tag pages, I added `hideTagPages: true` and `hideFolderPages: true` to my `quartz.config.yaml` under the recent notes plugin. Here is the resulting plugin setup:
`- source: "@quartz-community/recent-notes"`
`enabled: true`
`options:`
`title: Recent Posts`
`limit: 5`
`linkToMore: false`
`showTags: true`
`hideTagPages: true`
`hideFolderPages: true`
Make sure to save and `npx quartz sync` to push your changes.

To fix the 404 page, there is no built-in exclude setting in the plugin, so I had to do some custom CSS. On your working Linux CLI:

1. Navigated to my Quartz repository
2. Entered `nano quartz/styles/custom.scss`
3. Added the following to my config:

`.recent-notes li:has(a[href$="404"]) {`
`display: none;`
`}`

Your resulting CSS should look like this:

`@use "./variables.scss" as *;`

`// put your custom CSS here!`
`.recent-notes li:has(a[href$="404"]) {`
`display: none;`
`}`

4. Pressed **CTRL+X** | **Y**
5. Entered `npx quartz sync`

To explain the CSS, the `.recent-notes` addresses the recent notes plugin. `li` addresses list items within the recent notes plugin, each post in recent notes is a list item. `:has()` picks something from the list items, regarding what is inside the parenthesis. `a[href$="404"]` is pointing to a link (`a`) in a list item that references (`href`) a page that ends with (`$=`) a 404 (`"404"`). `display: none;` is saying to not display the 404 page that was chosen.  With this, your 404 page will still exist and be accessible when needed, but will no longer be displayed under your recent notes.

## Mobile view

While I am still trying to figure out how to optimize the mobile view, I found one of the biggest culprits of the mess to be the recent notes plugin. In order to still have it display on desktop, but exclude from mobile, I added `display: desktop-only` to my `quartz.config.yaml` under the recent notes plugin in its `layout:` section. The resulting plugin configuration looks like this:

`- source: "@quartz-community/recent-notes"`
`enabled: true`
`options:`
`title: Recent Posts`
`limit: 5`
`linkToMore: false`
`showTags: true`
`hideTagPages: true`
`hideFolderPages: true`
`layout:`
`position: left`
`priority: 25`
`display: desktop-only`

# Sources

[Useful Video Tutorial](https://www.youtube.com/watch?v=6s6DT1yN4dw)
[Quartz Documentation](https://quartz.jzhao.xyz/)
[Cloudflare Pages Documentation](https://developers.cloudflare.com/pages/)
[Syncthing Getting Started Documentation](https://docs.syncthing.net/intro/getting-started.htmlhttps://docs.syncthing.net/intro/getting-started.html)
[Quartz Syncer Documentation](https://saberzero1.github.io/quartz-syncer-docs/)
[Giscus Repository](https://github.com/giscus/giscus)
