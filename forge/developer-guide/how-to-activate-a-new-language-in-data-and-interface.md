<!--
parent: 'Developer Guide'
created_at: '2014-12-10 10:09:23'
updated_at: '2015-07-28 08:49:53'
authors:
    - 'Vijai Pandey'
tags:
    - 'Developer Guide'
-->

How to activate a new data and interface language
=====================================================

Here are the steps to Enable a new language for Data and Interface.

Step-1 : Start your command line interface and Switch to the {path_to_your_tao_installation}/tao/scripts on my mac, it is like this
---------------------------------------------------------------------------------------------------------------------------------------

**cd /Applications/XAMPP/xamppfiles/htdocs/tao/tao/scripts**

Step-2: Create locale folder and Necessary files for ‘tao’ extension (which is meta extension)
----------------------------------------------------------------------------------------------

**php taoTranslate.php -v -a=create -e=tao -l=hi-IN -ll=“Hindi”**

(VERY IMPORTANT: your language code should be 4 letter only. Starting with two small letters e.g. ‘hi’ for Hindi followed by a hyphen (-) and then two letter code for the country e.g. IN for India. If you use three letter location identifier like ‘ind’ for India or ‘arb’ for Arab World it will have problem later with item preview.)

Create similar folder and files for all other extensions (not all extensions are listed here)

php taoTranslate.php -v -a=create -e=filemanager -l=hi-IN -ll=“Hindi”<br/>

php taoTranslate.php -v -a=create -e=ltiDeliveryProvider -l=hi-IN -ll=“Hindi”<br/>

php taoTranslate.php -v -a=create -e=taoItems -l=hi-IN -ll=“Hindi”<br/>

php taoTranslate.php -v -a=create -e=taoQtiItem -l=hi-IN -ll=“Hindi”<br/>

php taoTranslate.php -v -a=create -e=taoQtiTest -l=hi-IN -ll=“Hindi”<br/>

php taoTranslate.php -v -a=create -e=taoQtiCommon -l=hi-IN -ll=“Hindi”<br/>

php taoTranslate.php -v -a=create -e=taoTests -l=hi-IN -ll=“Hindi”

take the .po files from the local/{your language} folder within these extensions. Translate them and and replace the translated files with old files.

Step-3: Run the compile command to compile all files for each extension..
-------------------------------------------------------------------------

**php taoTranslate.php -v -a=compile -e=tao -l=hi-IN**

**php taoTranslate.php -v -a=compile -e=taoSubjects -l=hi-IN**

**php taoTranslate.php -v -a=compile -e=taoQtiItems -l=hi-IN**

only 3 are given here, you have to run this command for all extensions.

Step-4: Reinstall TAO (Imp: do not forget to take backup before reinstalling)
=============================================================================

