---
tags: Forge
---

Updating Extensions from 2.6 to 3.0
===================================

This is work in progress.

Steps required:

-   Autoloading has been delegated to composer, please add a composer.json to your extension with the required autoload directives. Please consider that these only get registered after you installed your extension using composer.

<!-- -->

-   Update the structures.xml file from the 2.6 to the [[Tao 3.0 structures.xml format]]

<!-- -->

-   “New Item” is no longer shared by all item types, but has been moved to taoQtiItem. Custom item types need to add their own “add Item” action.
