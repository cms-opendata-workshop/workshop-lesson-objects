---
title: Setup
---
Follow the pre-exercises and get into a docker container or VM. 

The AOD2NanoAODOutreachTool repository is the example we will use for accessing information from AOD files. 
Check out a "dummy" version for doing workshop exercises. For reference, https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool 
is the official version of this code. 

~~~
$ cd ~/CMSSW_5_3_32/src/
$ cmsenv
$ mkdir workspace
$ cd workspace
$ git clone -b dummyworkshop git://github.com/jmhogan/AOD2NanoAODOutreachTool.git 
$ cd AOD2NanoAODOutreachTool
$ scram b
~~~
{: .language-bash}

{% include links.md %}
