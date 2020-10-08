---
layout: lesson
root: .  # Is the only page that doesn't follow the pattern /:path/index.html
permalink: index.html  # Is the only page that doesn't follow the pattern /:path/index.html
---

When a physicist approaches an analysis using CMS data, they typically rely on the reconstruction
algorithms developed by CMS to interpret detector signals into meaningful **physics objects**. In
code, the result of these recontruction algorithms takes the form of several C++ classes that 
will be introduced briefly in this lesson. The content of the C++ class reflects the nature of the
physics object it represents. 

In this lesson we will study several fundamental particles: muons, electrons, photons, and tau
leptons. The first three particles are special in CMS, because they are reconstructed as single 
**"particle-flow candidates"**. The Particle Flow algorithm (CITE ME) combines detector signals 
from multiple CMS subdetector systems to categorize all energy deposits as muons, electrons, 
photons, neutral hadrons, or charged hadrons. Tau leptons are more complex because they are not
stable and have several detector signatures that include muons, electrons, photons, and/or 
hadrons. In the next lesson we will approach even more complex objects such as jet and missing 
transverse energy. 

After exploring the code elements that are common to all CMS physics objects we will look at
muons, electrons, photons, and tau leptons in more detail to understand the options for 
identifying these particles in your analysis. The final episode (MAYBE IN SELECTION LESSON?) will
show how an analyzer can combine different identification elements into selection criteria.

<!-- this is an html comment -->

{% comment %} This is a comment in Liquid {% endcomment %}

> ## Prerequisites
>
> Complete the pre-exercises, watch the objects demonstration video, and **follow the instructions one the setup page**.
{: .prereq}

> ## Big questions
> 
> 1. What are the main CMS "physics objects" and how do I access them?
> 2. How can I store basic information about a given object, like 4-vectors?
> 3. Where can I find CMS-specific information about a given object, like identification criteria?
> 4. How do I select events based on these objects?
>
{: .checklist}

{% include links.md %}
