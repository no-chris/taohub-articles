---
tags: Forge
---

Installation Solr MacOs
=======================

To install Solr you can either download from http://www.apache.org/dyn/closer.cgi/lucene/solr/ or use Homebrew ( http://brew.sh/ ) packet manager

brew install solr

To have launchd start solr at login:

ln -sfv /usr/local/opt/solr/\*.plist \~/Library/LaunchAgents
