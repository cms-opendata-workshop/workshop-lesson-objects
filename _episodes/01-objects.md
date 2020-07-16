---
title: "CMS Physics Objects"
teaching: 10
questions:
- "What are the main CMS Physics Objects and how do I access them?"
objectives:
- "Identify CMS physics objects"
- "Identify object code collections in AOD files"
- "Access an object code collection"
keypoints:
- "CMS physics objects include: muons, electrons, taus, photons, and jets." 
- "Missing transverse momentum is derived from physics objects (negative vector sum)."
- "Objects are stored in separate collections in the AOD files"
- "Objects can be accessed one-by-one via a for loop"
---

CMS uses the phrase "physics objects" to speak broadly about particles that can be identified via 
signals from the CMS detector. A physics object typically began as a single particle, but may have 
showered into many particles as it traversed the detector. The principle physics objects are:

*   muons
*   electrons
*   taus
*   photons
*   jets

## Viewing collections

CMS AOD files store all physics objects of the same type in "collections". These collections are 
data structures that act as containers for multiple instances of the same C++ class. As an example,
the "selectedMuons" collection in AOD contains many pat::Muon objects (one per muon in the event).

~~~
$ edmDumpEventContent testfile.root
~~~
{: .source}

~~~
std::vector<pat::Muon> selectedMuons BlahBlah BlahBlah BlahBlah
std::vector<other::stuff> MoreLinesOfOutput
~~~
{: .output}

As you can see, the AOD file contains MANY collections, and not all of them are related directly to 
physics objects. Other important collections might include:

*   Missing transverse momentum (derived from a negative vector sum of physics objects)
*   Primary vertices
*   Secondary vertices (decays of particles produced at the primary vertex)
*   Triggers
*   Pileup (info about multiple collisions from the same pp crossing)
*   etc...

## Opening a collection

The [AOD2NanoAODOutreachTool](https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool) 
repository is the example we will use for accessing information from AOD files. 

The definitions of the muon classes are included:
~~~
#include "DataFormats/MuonReco/interface/Muon.h"
#include "DataFormats/MuonReco/interface/MuonFwd.h"
#include "DataFormats/MuonReco/interface/MuonSelectors.h"
~~~
{: .source}

You learned about the EDAnalyzer class in the pre-exercises. The AOD2NanoAOD tool is an EDAnalyzer. 
The "analyze" function of an EDAnalyzer is performed once per event. Muons can be accessed like this:

~~~
void AOD2NanoAOD::analyze(const edm::Event &iEvent,
                          const edm::EventSetup &iSetup) {

  using namespace edm;
  using namespace reco;
  using namespace std;

  Handle<MuonCollection> muons;
  iEvent.getByLabel(InputTag("muons"), muons);
~~~ 
{: .source}

The result is an  called "muons" which is a collection of all the muon objects. 
In the next episode we'll look at the member functions for muons.
Collection classes are generally constructed as std::vectors. We can 
quickly access the number of muons per event and create a loop to access 
individual muons:

~~~
int nMuons = muons->size();
for (auto it = muons->begin(); it != muons->end(); it++) {
    if (it->pt() > mu_min_pt) {
        // do things here, next episode!
    }
}
~~~
{: .source}


{% include links.md %}

