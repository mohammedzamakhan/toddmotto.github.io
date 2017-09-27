---
layout: post
permalink: /things-i-use
title: Things I use, desk, software, setups
path: 2016-10-06-things-i-use.md
---


I often get asked what colour schemes I'm using, stuff for videos, workshops, slides and so forth. So this is essentially a quick write up on that stuff.

<div class="toc" markdown="1">
<span class="gamma">Table of contents</span>
{:.no_toc}
* TOC
{:toc}
</div>

### Editor / Terminal

I don't have a typical engineering job as such as I [build courses](https://ultimateangular.com) and projects for teaching people Angular, therefore my workflow isn't as likely refined as someone who's heads down engineering full-time. However, prior to going solo on Ultimate Angular, I was a full-time engineer for around 6-7 years and used a similar workflow/setup:

* [VSCode Editor](https://code.visualstudio.com)
  * Plugin: [angular2-inline](https://marketplace.visualstudio.com/items?itemName=natewallace.angular2-inline)
  * Plugin: [vscode-icons](https://github.com/vscode-icons/vscode-icons)
* [Operator Mono](http://www.typography.com/blog/introducing-operator) font

VSCode `settings.json` config:

{% highlight json %}
{
  "workbench.welcome.enabled": false,
  "terminal.integrated.shell.osx": "/bin/zsh",
  "window.zoomLevel": 1,
  "editor.fontFamily": "Operator Mono",
  "editor.fontSize": 12,
  "editor.fontWeight": "100",
  "editor.tabSize": 2,
  "editor.cursorBlinking": "smooth",
  "typescript.check.tscVersion": false,
  "explorer.openEditors.visible": 0,
  "editor.quickSuggestions": true,
  "editor.suggestOnTriggerCharacters": true,
  "files.exclude": {
    "**/.awcache": true,
    "**/.vscode": true,
    "**/node_modules": true,
    "**/.gitignore": true,
    "**/.editorconfig": true
  },
  "editor.renderIndentGuides": false,
  "editor.minimap.enabled": false,
  "editor.tabCompletion": true,
  "vsicons.dontShowNewVersionMessage": true,
  "workbench.colorTheme": "Visual Studio Dark",
  "workbench.iconTheme": "vscode-icons",
  "vsicons.projectDetection.autoReload": true,
  "prettier.singleQuote": true,
  "prettier.useTabs": false,
  "prettier.tabWidth": 2,
  "prettier.printWidth": 80,
  "prettier.trailingComma": "es5",
  "editor.formatOnSave": true,
  "html.format.enable": false,
  "workbench.startupEditor": "newUntitledFile"
}

{% endhighlight %}

* I use VSCode's integrated terminal over iTerm now.
* For Git stuff I use [SourceTree](https://www.sourcetreeapp.com/) because I suck at command-line Git :)

That's about it, I'm all for keeping it simple.

### Conference slides

* [Spectacle Boilerplate](https://github.com/FormidableLabs/spectacle-boilerplate) to create all my slides in React/JSX
* Alongside Spectacle Boilerplate I run [Spectacle Codeslide](https://github.com/thejameskyle/spectacle-code-slide) for interactive code walkthroughs

### Desk and computer

* Desk: [VariDesk Standing Desk](https://www.varidesk.com/products/full-standing-desk-pro-desk?o=Size:60+inch;Color:Black) standing desk (huge workflow/comfort changer)
* Chair: [Humanscale Freedom](https://uk.humanscale.com/products/product.cfm?group=FreedomTaskChairWithHeadrest)
* Laptops/Desktops: MacBook Pro 15" touchbar (primary machine), MacBook Pro 15" retina x2, MacBook 12", iMac 27" (I use only one MacBook with an external monitor)
* Monitor: [34" curved LG](https://www.amazon.co.uk/dp/B01BFH1AIM/ref=pe_385721_137066351_TE_dp_1) 21:9 ultrawide (this has completely changed my working style, 100% recommend)
* Apple Time Capsule for backups
* Sound system: [Bose 50](https://www.amazon.co.uk/dp/B00UF6XF2E/ref=pe_1909131_77697001_tnp_email_TE_AMZLdp_1)

### Screencasts

* Microphone: Currently using the [Yeti](http://www.bluemic.com/products/yeti/) with a popshield
* [Screenflow](http://www.telestream.net/screenflow/overview.htm) for recording and editing videos

### Other

* Spotify is my life long companion in getting stuff done :)
* Slack for procrastinating my life away ;)

That's my entire setup! I keep it simple and easy.
