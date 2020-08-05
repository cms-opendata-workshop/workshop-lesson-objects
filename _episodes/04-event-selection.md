---
title: "Event Selection"
teaching: 15 min
exercises: 15 min
questions:
- "How do I select events based on physics objects?"
.objectives:
- "Understand typical object selection criteria"
- "Implement selections for the muon and tau examples"
- "Begin the workshop's physics analysis example"
.keypoints:
- "Selection criteria include kinematic limits (momentum and angle), identification, and isolation."
- "Kinematic limits are usually based on detector ranges and the physics process being studied."
- "Identification and isolation criteria depend mostly on your physics analysis goals."
---



## 

CMS NanoAOD files store per-event information that is needed in analyses. 
NanoAOD files are stored in Ntuple format and contains a main TTree named Events and some additional TTrees for run, lumi and metadata. 

In the previous exercises, you learned how to access and store object information from an AOD file and convert the AOD file to NanoAOD using the [AOD2NanoAODOutreachTool](https://github.com/cms-opendata-analyses/AOD2NanoAODOutreachTool). 
[HiggsTauTauNanoAODOutreachAnalysis] (https://github.com/cms-opendata-analyses/HiggsTauTauNanoAODOutreachAnalysis) repository  contains an analysis that reduces the NanoAOD files to study the decay of a Higgs boson to two tau leptons.
This exercise uses this repository as the example we will use for selecting events in the NanoAOD based on requirements on muons and taus.  

Data samples to be used in the analysis:

*Run2012B_TauPlusX
*Run2012C_TauPlusX


Simulations to be used in the analysis:

*GluGluToHToTauTau
*VBF_HToTauTau
Background processes can produce very similar signatures in the detector which have to be considered in the anaylsis.
The most prominent background processes with a similar signature:
*DYJetsToLL
*TTbar
*W1JetsToLNu
*W2JetsToLNu
*W3JetsToLNu

We have to take all these background processes into account when analyzing our data and drawing conclusions.
In event selection, our goal is to keep the signal coming from Higss to tau tau decay and suppress the backgrounds.
These background processes can be suppressed by introducing kinematic limits on the physics objects.
For instance, the W boson can decay into a lepton. The leptons can be misidentified as coming from a signal. 
A cut in the event selection on the transverse mass of the muon and the missing energy can strongly suppress this background coming from W+ jets. 

#Viewing NanoAOD files

~~~
root -l root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/GluGluToHToTauTau.root
new TBrowser;
~~~
{: .source}

[Insert picture of Tree here]

{: .output}

You can now view the Event branches by selecting the root file GluGluToHToTauTau.root and selecting the main TTree named Events.
You can make cuts interactively on event branches in root.
As an example there is a branch named Muon_pt and we can make the event selection for Muon pt > 17 

~~~
TTree *Events = (TTree*)_file0->Get("Events")
Events->Draw("Muon_pt","Muon_pt>17")
~~~
{: .source}

[Insert output]
{: .output}

The RDataFrame in ROOT offers a high level interface for anlyses of data stored in TTree, 
~~~
#include "ROOT/RDataFrame.hxx"
ROOT::EnableImplicitMT(); // Tell ROOT you want to go parallel
ROOT::RDataFrame df("Events", "root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/GluGluToHToTauTau.root"); //Interface to TTree
auto myHisto = df.Histo1D("Muon_pt"); // This happens in parallel!
myHisto->Draw();
~~~
{: .source}

[Insert output]
{: .output}

The RDataFrame can perform transformations (to manupulate the data) and actions (to produce a result from the data). 
The Define and Filter methods are transformations while the Count and Report methods are actions.

Define - Creates a new column in the datset 
Filter - Filters the rows of the dataset
Count  - Returns the number of events processed
Report - Obtains statistics on how many entries have been accepted and rejected by the filters

~~~
std::cout << "Number of events: " << *df.Count() << std::endl;
~~~

Source: (https://root.cern/doc/master/classROOT_1_1RDataFrame.html)

This function performs a selection on the minimal requirements of an event.
Here we require that the event passes a high level trigger and we have at least one muon and tau candidate in our event.
 
~~~
template <typename T>
auto MinimalSelection(T &df) {
    return df.Filter("HLT_IsoMu17_eta2p1_LooseIsoPFTau20 == true", "Passes trigger")
             .Filter("nMuon > 0", "nMuon > 0")
             .Filter("nTau > 0", "nTau > 0");
}
~~~
{: .source}

We use this funciton to find the interesting muons in the muon collection. 
This selection requires muons to have eta<2.1 and pt>17 and also requires the muon to pass tightId.

~~~
template <typename T>
auto FindGoodMuons(T &df) {
    return df.Define("goodMuons", "abs(Muon_eta) < 2.1 && Muon_pt > 17 && Muon_tightId == true");
}

~~~
{: .source}

We use this function to find the interesting taus in the tau collection. These tau candidates represent hadronic decays of taus which means that
the tau decays to combinations of pions and neutrinos in the final state. Add your cuts on tau eta and pt similar to how it was done for muon.

Also add requirements for the tau charge, Tau_idDecayMode, Tau_idIsoTight, Tau_idAntiEleTight, Tau_idAntiMuTight.

Notice that we have added an isolation requirement for our tau. 
Hard processes produce large angles between the final state partons. The final object of interest will be separated from 
the other objects in the event or be isolated. For instance an isolated muon from a W. In contrast, a non-isolated muon can come from
a weak decay inside a jet. Isolation variables are ET and pT sums in cones (drawn around the object) in eta-phi space. 
Isolation of a muon is done relative to detector objects such as detector hits and tracks.
~~~
template <typename T>
auto FindGoodTaus(T &df) {
    return df.Define("goodTaus", "PUT YOUR SELECTIONS HERE");
}
~~~
{: .source}

Implement the selections on muon and tau in a copy of the HiggsTauTauNanoAODOutreachAnalysis repository provided for you.

 
We can reduce the dataset to the interesting events containing at least one interesting
muon and tau candidate.

~~~
template <typename T>
auto FilterGoodEvents(T &df) {
    return df.Filter("Sum(goodTaus) > 0", "Event has good taus")
             .Filter("Sum(goodMuons) > 0", "Event has good muons");
}
~~~
{: .source}

We can then use the functions we have created to perform event selections on the GluGluToHToTauTau sample.

~~~
ROOT::EnableImplicitMT();
ROOT::RDataFrame df("Events", "root://eospublic.cern.ch//eos/opendata/cms/derived-data/AOD2NanoAODOutreachTool/GluGluToHToTauTau.root");
auto df2 = MinimalSelection(df);
auto df3 = FindGoodMuons(df2);
auto df4 = FindGoodTaus(df3);
auto df5 = FilterGoodEvents(df4);
~~~

~~~
auto dfFinal = df5;
auto report = dfFinal.Report();
dfFinal.Snapshot("Events", sample + "Skim.root", finalVariables);
report->Print();
~~~
{: .source}
