# BranchJira

Copies the right branch Name or a suitable begin of a Commit Message directly from Jira into your Clipboard

## Build

`npm install -g web-ext`

`web-ext build --overwrite-dest`

## Install

This is currently not signed, so to install in firefox you need a "blueisch" Firefox like the developer Edition

Additional you need to change the value of `xpinstall.signatures.required` in `about:config` to `false`

Just drag the zip located under web-ext-artifacts/ into `about:addons`

## Future

Maybe make a button appear on the jira site to do this things located in the popup
